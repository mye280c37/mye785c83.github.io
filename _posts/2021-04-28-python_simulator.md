---
layout: post
title : python을 이용한 간단한 network simulation 구현
category: AS818
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
## Mobile Node Deployment
<br>

### Skewed Deployment

AP와 ms 사이의 distance 값이 특정 범위에 몰리도록 mobile node를 분포하는 것

⇒ high demand node의 발생이 불규칙적이고 예측할 수 없을 때 특정 distance에 demand가 과도하게 발생한다면 논문에서 제시한 방법이 비효율적일 수 있다.


<div class="comment-box">
<strong>WHY?</strong><br>  
<p style="padding-left: 10px; margin-top: 1rem">
한 channel에 할당된 node들 사이에 distance차가 목표한 바와 다르게 커질 수 있기 때문
</p>
</div>

<br><br>


## Setting channel and CST
<br>

### 기존 base simulator logic

1. channel 할당
   * mobile node index를 `param_num_channels` 로 나눈 나머지

2. cst 설정
    * `param_cs_thresh` + `param_margin` 으로 모든 채널이 동일

### distance에 따른 node grouping

1. channel 할당
    * 각 channel은 `int(param_num_nodes / param_num_channels)` 만큼의 node를 할당받는다
    * node는 associated AP와의 distance 값이 작은 순서대로 channel index 0부터 차례대로 channel 할당을 받는다
        + channel index가 가장 작은 channel( $ch[0]$ )에 할당된 node들의 distance가 가장 작다
  
2. cst 설정
   * channel에 할당된 node들 중 distance값이 가장 큰 node 기준
   * 해당 node가 견딜 수 있는 SNR 크기를 경계로 하여 CST 설정

### channel에 할당된 node 개수에 따른 channel bandwidth 설정

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
