---
layout: post
title : python network simulator 분석
category: ICSL
tags: [python, network, wirelessNetwork]
use_math: true
complete: true
---

## RunSim parameter

``` python
RunSim(NUM_GRID=g, AREA_SIZE=a, NUM_NODES=n, NUM_CH=c, PROTO=p, CS=cs, SEED=s,
                                 END=end, VERBOSE=True, IDEAL=False, LATEX=False, 
                                 MARGIN=m)
```

parameter | - 
| :---: | :---: 
$NUM\text{_}GRID$ | GRID 개수, 시뮬레이션 환경 영역이 NUM_GRID*NUM_GRID로 나뉜다.
$AREA\text{_}SIZE$ | GRID 하나 크기
$NUM\text{_}NODES$ | node (mobile station) 개수
$NUM\text{_}CH$ | channel 개수
$PROTO$ | <span class="warning">(int)</span> protocol type
$CS$ | constant carrier sense threshold
$SEED$ | random seed
$END$ | simulation end time (반복 횟수)
$MARGIN$ | CST MARGIN (cst = $CS$ + $MARGIN$)
{:.math-table} 

#### PROTO

type | -
| :---: | :---: 
0 | carriersense
1 | DSC
2 | smart
3 | new carrier
{:.math-table}


## simulation logic

#### simulation 가정

1. SNR threshold만 넘으면 패킷이 성공적으로 수신되었다고 가정한다.
2. 설정된 END_TIME동안 TIMESLOT_PER_TX 을 주기로 하는 서브 시뮬레이션이 반복된다.
3. channel이 완전히 비어있는 상태에서 서브 시뮬레이션이 시작된다.

    ⇒ 따라서 처음에 destination이 있는 모든 ap가 channel이 idle이라고 감지한다.

4. TIMESLOT_PER_TX 동안 전송이 시작된 packet은 전송 시작된 시점에 상관없이 다음 iteration에서 모두 전송을 완료한다고 가정한다.


#### simulation flowchart

![simulation flowchart](/assets/images/simulation_flowchart.jpg)


- t에 대한 한번의 iteration에서 각 channel은 `time_slot_per_tx` 동안 simulation을 진행한다.
- destination을 가진 AP들 중 가장 작은 contention window(`win_count`)를 가진 AP가 packet 전송을 시작한다.
  - destination을 가진 AP들은 모두 channel이 idle이라고 판단하고 contention window count를 시작한 것
  - `win_count`만큼 모든 AP들이 contention window count를 진행한 것
- 나머지 destination을 가진 AP들은 carrier sensing을 통해 busy로 감지될 경우 transmission 시도를 pause한다.
- channel을 idle로 탐지한 AP들의 경우 자신의 남은 contention window에 대해 다시 count를 시작한다.
- 이 중 가장 작은 contension window를 가진 AP가 packet 전송을 시작한다.
- 누적된 `win_count` (`slot_count`)가 `time_slot_per_tx` 커지거나 더 이상 전송을 시도하는 AP가 없을 경우, 해당 iteration은 종료되고 전송을 시작한 packet들에 대해 SNR을 구해 packet이 성공적으로 도착했는지 판단한다.


## 앞으로 구현해야하는 코드

#### biased deployment 환경 구현

`biased_dist` 변수를 설정해 node의 1/3을 associated AP와의 distance가 특정 범위에 몰리도록 설정한다.


1. random하게 AP index를 고른다.
2. AP의 좌표로부터 node의 x, y좌표가 각각 `biased_dist` 에서 `biased_dist+1` 사이의 임의의 값만큼 떨어지도록 node의 좌표를 설정한다.


#### carrier sensing threshold 설정

node를 distance에 따라 균등하게 분배하거나 distance를 균등하게 분배한 후 node를 할당하는 방식의 simulation의 경우 carrier sensing threshold를 해당 채널에 할당된  node들 중 가장 큰 distance를 가진 node를 기준으로 carrier sensing threshold를 가질 수 있도록 코드를 구현한다.

1. 채널 할당 시, 거리 순으로 node index를 정렬한다.
2. 거리 순으로 node index를 돌며 채널을 0부터 할당할 때, 채널 index가 증가하는 시점에 가장 마지막에 해당 채널에 할당된 node index를 구한다.
3. 구한 node index가 자신의 associated AP와 transmission을 성공할 수 있는 `MinCSTThresh` 를 구해 해당 값을 channel cst로 정한다.



#### distance range 설정

associated AP와 비슷한 distance를 가진 node들을 같은 채널에 묶기 위해 각 채널에 distance range를 정한다.

전체 node의 distance 최솟값과 최댓값 사이의 범위를 channel 개수만큼 균등하게 나누어 node의 associated AP와의 거리가 channel의 distance range 안에 들어올 경우 node를 해당 채널에 할당한다.


#### channel bandwidth 설정

distance range 설정을 통해 채널에 할당된 node들의 개수에 비례하게 channel_weight 를 설정해 해당 값을 bandwidth로 사용  
=> 기준 값은 node 균등 분배시 각 채널에 할당되는 node 개수 (`param_num_node/param_num_channel`)


## 수정된 RunSim

```python
RunSim(NUM_GRID=g, AREA_SIZE=a, NUM_NODES=n, NUM_CH=c, PROTO=p, CS=cs, SEED=s,
                              END=end, VERBOSE=True, IDEAL=False, 
                              LATEX=False, MARGIN=m, DEPLOY=1, 
                              SET_CHANNEL=2, SET_CST=1, BIASED_DIST=biased)
```

parameter | - 
| :---: | :---: 
$NUM\text{_}GRID$ | GRID 개수, 시뮬레이션 환경 영역이 NUM_GRID*NUM_GRID로 나뉜다.
$AREA\text{_}SIZE$ | GRID 하나 크기
$NUM\text{_}NODES$ | node (mobile station) 개수
$NUM\text{_}CH$ | channel 개수
$PROTO$ | <span class="warning">(int)</span> protocol type
$CS$ | constant carrier sense threshold
$SEED$ | random seed
$END$ | simulation end time (반복 횟수)
$MARGIN$ | CST MARGIN (cst = $CS$ + $MARGIN$)
<span style="color:blue">$DEPLOY$</span> | <span class="warning">(int)</span> <span style="color:blue">deployment type</span>
<span style="color:blue">$SET\text{_}CHANNEL$</span> | <span class="warning">(int)</span> <span style="color:blue">channel setting type</span>
<span style="color:blue">$SET\text{_}CST$</span> | <span class="warning">(int)</span> <span style="color:blue">cst standard type</span>
<span style="color:blue">$BIASED\text{_}DIST$</span> | <span style="color:blue">특정하게 몰릴 distance 값</span>
{:.math-table} 


#### DEPLOY

type | -
| :---: | :---: 
0 | random deployment
1 | biased deployment
{:.math-table}


#### SET_CHANNEL

type | -
| :---: | :---: 
0 | random setting
1 | node의 associated AP와의 distance에 따른 균등 분배
2 | 균등한 distance range 설정에 따른 node 분배 및 node 개수에 비례하는 channel bandwidth 설정
{:.math-table}


#### SET_CST

type | -
| :---: | :---: 
0 | 모든 node가 `RunSim` parameter로 넘어온 cst 값을 가짐
1 | cst설정의 기준 node를 해당 채널에 할당된 node들의 end node로 설정
2 | cst설정의 기준 node를 해당 채널에 할당된 node들의 middle node로 설정
3 | <span class="warning">(SET_CHANNEL=2인 경우에만 유효)</span> cst를 SET_CHANNEL=1 인 경우와 동일하게 설정
{:.math-table}