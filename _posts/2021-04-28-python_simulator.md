---
layout: post
title : python을 이용한 간단한 network simulation 구현
category: AS818
tags: [python]
use_math: true
---

#### variable setting
```python
'''
[input]
----------------------------------------------------------------
NUM_GRID,       grid 개수. NUM_GRID=n에 대해 n*n개의 AP가 생성됨
AREA_SIZE,      한 grid 당 사이즈
NUM_NODES,      mobile node 개수
NUM_CH,         channel 개수
PROTO,          protocol(tramsmission) type 
    0: carriersense
    1: DSC
    2: smart
    3: new carrier
CS,             CST 값 
SEED,           seed 값
END,            simulation end time. END만큼 각 채널이 전송 반복
VERBOSE, 
IDEAL, 
LATEX, 
MARGIN,         cst margin 값 (실제 setting cst = 설정 cst +  margin)
DEPLOY,         mobile node 분포 형태
    0: seed를 이용한 random 분포
    1: AP와 ms 사이의 distance가 특정 범위에 몰리도록 분포
SET_CHANNEL,
    0: 기존의 방식대로
    1: 각 채널이 distance에 따라 같은 개수의 node를 가지도록
    2: 각 채널이 assigned node 개수에 따라 다른 bandwidth를 가지도록
------------------------------------------------------------------
'''

def RunSim(NUM_GRID, NUM_NODES, NODES_PER_CELL, AREA_SIZE, NUM_CH, CS, PROTO, SEED ,END, VERBOSE, CH_BW_LOSS, LATEX, IDEAL, MARGIN, TOKEN, DEPLOY,SET_CHANNEL):
    # variable setting
    param_seed = SEED
    param_num_grid = NUM_GRID
    param_num_aps = param_num_grid * param_num_grid
    param_num_nodes = NUM_NODES
    param_num_channels = NUM_CH
    param_end_time = END
    param_cs_thresh = CS
    param_protocol = PROTO
    param_verbose = VERBOSE
    param_ch_bw_loss = CH_BW_LOSS
    param_export_latex = LATEX
    param_margin = MARGIN
```

## 1차
<br><br>

### Mobile Node Deployment
<br>

#### Skewed Deployment

AP와 ms 사이의 distance 값이 특정 범위에 몰리도록 mobile node를 분포하는 것

⇒ high demand node의 발생이 불규칙적이고 예측할 수 없을 때 특정 distance에 demand가 과도하게 발생한다면 논문에서 제시한 방법이 비효율적일 수 있다.


<div class="comment-box">
<strong>WHY?</strong><br>  
<p style="padding-left: 10px; margin-top: 1rem">
한 channel에 할당된 node들 사이에 distance차가 목표한 바와 다르게 커질 수 있기 때문
</p>
</div>

<br><br>


### Setting channel and CST
<br>

#### type1. 기존 base simulator logic

1. channel 할당
   * mobile node index를 `param_num_channels` 로 나눈 나머지

2. cst 설정
    * `param_cs_thresh` + `param_margin` 으로 모든 채널이 동일

#### type2. distance에 따른 node grouping

1. channel 할당
    * 각 channel은 `int(param_num_nodes / param_num_channels)` 만큼의 node를 할당받는다
    * node는 associated AP와의 distance 값이 작은 순서대로 channel index 0부터 차례대로 channel 할당을 받는다
        + channel index가 가장 작은 channel( $ch[0]$ )에 할당된 node들의 distance가 가장 작다
  
2. cst 설정
   * channel에 할당된 node들 중 distance값이 가장 큰 node 기준
   * 해당 node가 견딜 수 있는 SNR 크기를 경계로 하여 CST 설정

#### type3. channel에 할당된 node 개수에 따른 channel bandwidth 설정

1. channel 할당
    * 각 node는 associated AP와의 distance가 작은 순서대로 channel index 0부터 차례로 channel 할당을 받는다.
    * channel에 할당된 node의 dist중 가장 작은 dist( `min_dist` )를 기준으로 `dist_threshold`를 정해 해당 할당 받는 node의 distance가 `min_dist + dist_threshold` 보다 클 경우 channel index를 1 늘린다.
    * 할당된 node 개수를 비율만큼 channel bandwidth를 할당한다.
       + [simulator 상에서 구현]

            simulator는 1 time마다 각 channel을 돌면서 `tx_vector` 와 `rx_vector` 를 구한다.

            실제로 네트워크 상에서 bandwidth가 커지면 packet 전송 속도가 빨라지기 때문에 1 time당 더 많은 packet이 오갈 수 있다.

            simulator 상에서는 bandwidth에 따라 `rx_vector` 에 weight를 주어 속도를 주었다.

            ( » *weight: `int(param_num_nodes / param_num_channels)` 와의 비율* )
2. cst 설정
   * channel에 할당된 node들 중 distance값이 가장 큰 node 기준
   * 해당 node가 견딜 수 있는 SNR 크기를 경계로 하여 CST 설정

3. `stat_rx_packets_ms` 카운트
   * 매 `t`, `ch`에 따른 `rx_vector * ch_weight[ch]`를 더함

<br>

### 결과 분석

1. random하게 분포된 node에서 $\text{type1}$과 $\text{type2}$의 throughput을 비교해보면 $\text{type2}$가 $1.25$ 배 가량 향상된 것을 확인할 수 있었다.
    + <span style="color: red;">*실제 논문에서는 2배 가량 차이나는데 어떤 차이일지?*</span>
  
2. skewed deployment에서 $\text{type2}$과 $\text{type3}$의 channel 별 throughput과 전체 throughput을 비교하면
   + 전체 throughput의 경우 거의 비슷하다(약간의 감소)
   +  channel별 throughput을 비교해보면 `ch_weight` 즉, bandwidth에 따라 설정된 $weight$ 가 크면 커지고 작으면 작아진다.
    
<br>

##### <2번 결과에 대한 분석>  

bandwidth가 클수록 그만큼 그 채널에 할당된 node가 많다는 뜻인데 그 채널에 대해서<span style="color: red;">'**만**'</span> $weight$에 의해 많은 throughput이 생성된 것처럼 되고 나머지 다른 채널의 경우 모두 $weight$에 의해 throughput이 낮아지기 때문에 $weight$가 큰 채널에 의해 높은 throughput을 가지게된 node와 $weight$가 낮은 채널에 의해 낮은 throupghput을 가지게된 node간의 throughput 상쇄가 일어나 결론적으로는 합이 비슷해지는 것 같다.

<br>

##### <2번 결과에 대한 대안>

실제로는 bandwidth가 커지면 그만큼 $\text{mbps}$ 가 커져서 한 timeslot에 더 많은 packet이 전송될 수 있다.(수신을 성공하는지와는 관계 없이)
따라서 성공한 packet에 $weight$를 부여하는 대신 timeslot 자체를 $weight$에 따라 조절해 bandwidth가 커 weight가 커지면 timeslot도 $\text{mbps}$가 커진 효과를 내보려 한다.

<br><br>

## 2차
<br>

$\text{type1}$, $\text{type2}$에 대한 setting은 유지

<br>

#### type3. channel에 할당된 node 개수에 따른 channel bandwidth 설정

1. channel 할당
   * 1차와 동일
  
2. cst 설정
   * 1차와 동일

3. `timeslot_per_tx` 설정
   * 기존의 `TIMESLOTS_PER_TX`로 고정되어 있던 time slot 크기를 `TIMESLOTS_PER_TX * ch_weight[ch]`로 변경
        + 기대 결과

            늘어난 timeslot만큼 $\text{tx_packet}$ 의 수가 늘어날 것으로 예상
        

        + 실제 결과
            
            실제로 $\text{type2}$와 결과를 비교해보면 $\text{tx_packet}$의 수는 줄어들고, $\text{lost_packet}$의 수는 거의 비슷해 결론적으로 $\text{total throughput}$이 줄어드는 것을 확인할 수 있었다.

        + 결과 분석

            각 채널에 할당된 cst를 확인해보면 미세하게 $\text{type3}$의 cst가 모두 작은 것을 확인할 수 있다.

            cst 할당 방법은 가장 멀리 있는 node를 기준으로 설정하는 방법을 택했고 skewed deployment 결과 앞쪽 그룹에 node들이 많이 몰려 있기 때문에 각 채널의 가장 멀리 있는 node의 $\text{distance}$ 값이 모두 증가한다. 따라서 cst 값이 작아질 수 밖에 없다.

            channel 별 throughput을 비교하면 모든 채널의 throughput이 감소하는데 `ch_weight`가 1 이하인 channel의 경우, timeslot이 감소해서인 것 같다.

            다만, `ch_weight`가 1보다 큰 경우에도 channel throughput이 감소하는데 이는 `ch_weight`에 의해 늘어난 timeslot의 이득보다 node 개수가 많아지면서 증가한 간섭에 의한 손실이 더 커서인 것으로 예상된다.