---
layout: post
title : python network simulator 분석
category: ICSL
tags: [python, network, wirelessNetwork]
use_math: true
complete: true
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

## simulation logic

#### simulation 가정

1. SNR threshold만 넘으면 패킷이 성공적으로 수신되었다고 가정한다.
2. 설정된 END_TIME동안 TIMESLOT_PER_TX 을 주기로 하는 서브 시뮬레이션이 반복된다.
3. channel이 완전히 비어있는 상태에서 서브 시뮬레이션이 시작된다.

    ⇒ 따라서 처음에 destination이 있는 모든 ap가 channel이 idle이라고 감지한다.

4. TIMESLOT_PER_TX 동안 전송이 시작된 packet은 전송 시작된 시점에 상관없이 다음 iteration에서 모두 전송을 완료한다고 가정한다.


#### simulation flowchart

![simulation flowchart](/assets/images/simulation_flowchart.jpg)


- 한번의 t iteration에서 각 channel은 `time_slot_per_tx` 동안 simulation을 진행한다.
- destination을 가진 AP들 중 가장 작은 contention window(`win_count`)를 가진 AP가 packet 전송을 시작한다.
  - destination을 가진 AP들은 모두 channel이 idle이라고 판단하고 contention window count를 시작한 것
  - `win_count`만큼 모든 AP들이 contention window count를 진행한 것
- 나머지 destiation을 가진 AP들은 carrier sensing을 통해 busy로 감지될 경우 transmission 시도를 pause한다.
- channel을 idle로 탐지한 AP들의 경우 자신의 남은 contention window에 대해 다시 count를 시작한다.
- 이 중 가장 작은 contension window를 가진 AP가 packet 전송을 시작한다.
- 누적된 `win_count` (`slot_count`)가 `time_slot_per_tx` 커지거나 더 이상 전송을 시도하는 AP가 없을 경우, 해당 iteration은 종료되고 전송을 시작한 packet들에 대해 SNR을 구해 packet이 성공적으로 도착했는지 판단한다.


## 앞으로 구현해야하는 코드

#### biased deployment 환경 구현


#### distance range 설정