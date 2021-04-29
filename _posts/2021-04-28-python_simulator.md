---
layout: post
title : python을 이용한 간단한 network simulation 구현
categories: [AS818]
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

## 기존 base simulator logic

1. channel 할당
   
   mobile node index를 `param_num_channels` 로 나눈 나머지

2. cst 설정

    `param_cs_thresh` + `param_margin` 으로 모든 채널이 동일

## distance에 따른 node grouping

1. channel 할당
    * 각 channel은 `param_num_channels`/`param_
2. cst 설정

## channel에 할당된 node 개수에 따른 channel bandwidth 설정

1. channel 할당
2. cst 설정