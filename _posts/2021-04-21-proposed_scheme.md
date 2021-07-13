---
layout: post
title : my proposed Scheme
category: ICSL
tags: [paper, network, wirelessNetwork]
use_math: true
complete: true
---

#### based paper

[A Simple and Practical Scheme Using Multiple Channels for Improving System Spectral Efficiency of Highly Dense Wireless LANs](https://www.hindawi.com/journals/wcmc/2019/2149606/)


## my proposed Scheme

network density가 높아지고 node의 이동성도 높아지는만큼 특정 distance를 가진 node들에 대한 traffic이 증가하는 traffic pattern이 발생할 수 있다. 


기존 base paper처럼 단순히 channel에 distance 순으로 같은 개수만큼 균등하게 node를 분배한다면 위와 같은 상황에서 한 채널을 할당받은 node group안에서 distance 차로 인한 edge node가 발생할 수 있다.  
또한 base paper에서 하나의 node group의 node들 중 가장 거리가 먼 node를 기준으로 CST를 설정하기 때문에 group 안에서 distance 차이가 크게 날 경우 conservative transmission이 심해질 수 있다.


따라서 distance 순으로 channel을 할당하는 것을 유지하되, distance가 비슷한 node들끼리 경쟁할 수 있도록 node를 grouping한 뒤, node 개수에 비례하게 channel bandwidth를 설정해 node들이 최대한 덜 conservative transmission을 할 수 있게 하면서 또 edge node의 발생을 줄일 수 있도록 한다.


#### my proposed Scheme에서 발생할 수 있는 문제 상황

network 환경이 특정 distance에 집중된 상황에서 bandwidth를 적용해 비슷한 distance끼리 경쟁하게 될 때 특정 distance에 너무 몰려버리면 비슷한 distance끼리 경쟁하게 돼서 얻을 수 있는 이점보다 너무 많은 node로 인해 collision 증가가 심해져 얻게 되는 손해가 더 커질 수 있다. 따라서 이 둘 사이에서 적당한 균형을 찾아야 한다.


## 1차 목표

associated AP와의 distance가 특정 범위에 몰리도록 설정된 환경에서 channel에 균등한 distance range를 설정해 해당 range 안에 들어오는 node를 channel에 할당하는 방식을 취한다.


이때 channel에 할당된 node 개수에 비례하게 bandwidth를 설정해 node 개수가 많은 channel의 경우 bandwidth가 그만큼 커져 더 많은 packet이 전송될 수 있도록 한다.


## 최종 목표

bandwidth가 넓어지면서 얻는 troughput 이득과 node 개수가 증가함에 따른 collision 증가로 인한 throughput 손해 사이에서 최대한 throughput을 높일 수 있는 방향을 찾도록 channel assignment logic을 설계한다. 