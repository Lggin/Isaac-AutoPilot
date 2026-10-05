# ROS 2 & Isaac Sim Autonomous Navigation

> **NVIDIA Isaac Sim + ROS 2 + Computer Vision** 기반 Ackermann 차량 자율주행 디지털 트윈 프로젝트

## Overview

NVIDIA Isaac Sim의 가상 환경과 ROS 2 Humble을 연동해 Ackermann 조향 차량의 자율주행 시스템을 구현했습니다.

전역 경로는 **A\*** 알고리즘으로 계산하고, 직진 구간에서는 OpenCV 기반 차선 인식, 교차로·회전 구간에서는 Odometry와 Yaw를 이용하는 하이브리드 주행 구조를 적용했습니다.

---

## System Architecture

```text
Isaac Sim
 ├─ RGB Camera ───────────────┐
 └─ Odometry ───────────────┐ │
                            ↓ ↓
                      ROS 2 Bridge
                            ↓
                 autonomous_nav_node
                 ├─ A* Global Planner
                 ├─ Vision Lane Tracking
                 ├─ Odom / Yaw Navigation
                 └─ Driving State Logic
                            ↓
                  /ackermann_cmd
                            ↓
                  Ackermann Vehicle
```

---

## Key Features

### 1. A* Global Path Planning

사전에 정의한 Map Graph의 node와 edge를 기반으로 현재 위치에서 목적지까지 최단 경로를 계산합니다.

### 2. Vision-based Lane Tracking

직진 구간에서는 카메라 영상을 받아 다음 과정을 수행합니다.

- HSV 기반 차선 색상 분리
- ROI 설정
- Image Moment로 차선 중심 계산
- 차량 중심과 차선 중심의 오차를 이용해 조향량 계산

### 3. Dual Navigation Mode

주행 구간의 성격에 따라 제어 방식을 분리했습니다.

- **VISION mode**: 카메라 기반 차선 추종
- **BLIND mode**: 교차로·회전 구간에서 Odometry/Yaw와 목표 방향 기반 조향

센서 하나에 모든 상황을 의존하지 않고, 구간 특성에 따라 적절한 정보를 사용하도록 상태 기반 주행 로직을 구성했습니다.

### 4. ROS 2 Interface

**Subscribers**
- `/camera_left/image_raw`
- `/odom`
- `/set_goal`

**Publishers**
- `/ackermann_cmd`
- `/camera_left/lane_overlay`

### 5. Real-time Route Visualization

`networkx`와 `matplotlib`을 사용해 현재 차량 위치와 A* 경로를 2D UI에 표시했습니다.

UI 갱신과 ROS 2 제어 루프가 서로 블로킹하지 않도록 실행 흐름을 분리해, 시각화가 주행 제어 주기를 방해하지 않도록 구성했습니다.

---

## My Contribution

- Isaac Sim ↔ ROS 2 Bridge 연동
- A* 기반 전역 경로 탐색 로직 구현
- OpenCV 기반 차선 인식 및 조향 오차 계산
- Odometry / Yaw 기반 교차로 주행 로직 구현
- VISION / BLIND 상태 전환 구조 설계
- 실시간 2D 경로 UI 구성 및 제어 루프와 실행 분리
- 전체 자율주행 흐름 통합 및 디버깅

---

## Tech Stack

`Ubuntu 22.04` `ROS 2 Humble` `NVIDIA Isaac Sim` `Python` `OpenCV` `NetworkX` `Matplotlib`

---

## Project Structure

```text
src/project/
├── project/
│   ├── line_detecing.py   # ROS 2 autonomous navigation node
│   └── map_car.py         # Isaac Sim map / vehicle loader
└── resource/
    ├── map.usd
    ├── ackermann_car_fixed_cam.usd
    └── assets/
```

---

## What I Learned

이 프로젝트를 통해 자율주행 시스템은 단일 알고리즘만으로 완성되지 않고, **경로계획·비전·위치추정·상태전이·실시간 제어**를 하나의 데이터 흐름으로 연결해야 한다는 점을 경험했습니다.

또한 시뮬레이션 환경에서도 ROS 2 토픽 주기와 UI 처리처럼 실제 시스템과 유사한 병목이 발생할 수 있어, 기능 구현뿐 아니라 실행 구조를 함께 설계해야 함을 배웠습니다.