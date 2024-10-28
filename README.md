
![image](https://github.com/user-attachments/assets/44f0aaa4-71b7-43af-9867-658f68c2598e)
![image](https://github.com/user-attachments/assets/1d08bd59-c086-4568-8329-6a8a0b2f5a57)
![image](https://github.com/user-attachments/assets/f54d1c03-2a0c-4a6b-9c1c-34896037ee1f)

# Graduation_Project_ws: Autonomous Stair-Climbing Mobility

**Graduation_Project_ws**는 계단을 오를 수 있는 자율주행 모빌리티 설계를 목표로 한 프로젝트입니다. 이 프로젝트는 모빈(MOBINN)의 자율주행 모빌리티를 모티브로 하여, ROS2 기반의 제어 및 Visual SLAM, 딥러닝 인지 알고리즘을 통해 계단을 오를 수 있는 모빌리티 시스템을 구축하였습니다. 






## 📌 프로젝트 개요

### 프로젝트 목표
1. 자율주행 모빌리티가 계단을 오르내릴 수 있는 기능 개발.
2. 실내외 환경에서 안정적인 위치 추정과 장애물 회피.
3. 특수 고무바퀴와 배선 설계, 모터 제어 및 Visual SLAM 활용.

### 주요 기능
- **실시간 계단 인식 및 자율 주행**: Depth Camera (D435i)를 활용하여 장애물 인식 및 계단 감지.
- **모터 및 전동실린더 제어**: Arduino 기반의 모터 및 실린더 제어로 계단을 오르고, 기울어진 상태에서 상체 수평 유지.
- **ROS2 기반의 데이터 송수신 및 제어**: ROS2 (Humble)를 활용하여 실시간 데이터 처리 및 제어.





  
---

## 📁 프로젝트 구조

```plaintext
Graduation_Project_ws/
├── src/
│   ├── stairs_detection/                          # 계단 인식 및 거리 계산 노드
│   │   └── dis_estimate.py                        # 딥스카메라로 계단 거리 계산
│   │   └── real_time_detection_closest.py         # 딥러닝 학습 모델 기반 실시간 계단 인식 및 거리 계산
│   ├── cmd_control/                               # 키보드 입력을 통한 제어
│   │   └── keyboard_controller.py                 # WASD 키보드 컨트롤러
│   ├── motor_control/                             # 아두이노 제어 코드
│   │   └── motor_control.ino                      # 모터 및 실린더 제어
│   └── mapping/                                   # RTAB-Map을 활용한 SLAM 노드
│       └── mapping_node.py                        # Visual SLAM 노드
├── README.md                                      # 프로젝트 개요
└── setup.py                                       # ROS2 패키지 설정
```






---

## 🛠️ 기술 스택 및 장비

- **ROS2 (Humble)**: ROS 기반 자율주행 구현
- **Intel RealSense DepthCamera D435i**: 계단과 주변 환경 인식, 경로 생성
- **RTAB-Map (Visual SLAM)**: 실내 Mapping 및 Localization
- **Arduino**: 모터 및 전동실린더 제어
- **Python & C++**: ROS2 노드 및 제어 코드

---





## 🛠️ 개발 과정




1. **딥스카메라 드라이버 활성화**
   - **명령어**: `ros2 launch realsense2_camera rs_launch.py`
   - **기능**: RealSense D435i 깊이 카메라 활성화로 장애물 검출 및 거리 계산 수행
   - **문제점**: 카메라가 38cm 이상의 거리에서 인식 성능 저하
   - **해결 방법**: 바운딩 박스 크기 비율을 이용하여 38cm 이상의 거리에서도 계단을 인식할 수 있도록 개선




2. **모빌리티 센서 및 TF 프레임 설정**
   - **명령어**: `ros2 run robot_state_publisher robot_state_publisher <경로>/minimo.urdf`
   - **결과**: URDF 파일을 통해 로봇의 센서 위치와 링크 정의를 정확하게 수행하여 tf 변환 설정




3. **계단 인식 노드 실행**
   - **명령어**: `ros2 run stairs_detection real_time_detection_closest`
   - **과정**: YOLOv5 모델을 사용해 계단 인식 후 깊이 카메라로 거리 계산 수행
   - **문제점**: 38cm 이상 거리에서의 인식 문제
   - **해결 방법**: 바운딩 박스 크기 비율을 사용하여 거리 추정값 보완




4. **SLAM 맵핑**
   - **명령어**: `ros2 launch rtabmap_launch rtabmap.launch.py`
   - **목적**: SLAM을 통해 실내 환경을 맵핑하여 로봇의 경로 계획과 장애물 회피 성능 강화
   - **문제점**: CPU만으로 처리 시 렉 발생
   - **해결 방법**: 프레임 수, 키프레임 간 이동 거리, 특징점 계산 수를 최적화하여 CPU 성능 개선




5. **SLAM 로컬라이제이션**
   - **명령어**: `ros2 launch rtabmap_launch rtabmap.launch.py localization:=true`
   - **결과**: 기존 맵 데이터를 활용한 로컬라이제이션 성능 강화
   - **문제점**: 특정 지점에서 정확도 저하
   - **해결 방법**: 반복적인 맵 취득으로 맵 세부 정보 보완 및 최적화




6. **맵 추출**
   - **명령어**: `rtabmap-databaseViewer ~/.ros/rtabmap.db`
   - **과정**: 맵 데이터를 `.pgm` 및 `.yaml` 형식으로 저장하여 네비게이션에 활용




7. **네비게이션 실행**
   - **명령어**: `ros2 launch nav2_bringup bringup_launch.py map:=~/.ros/rtabmap.yaml`
   - **문제점**: 프레임 및 tf 변환 설정 문제로 초기 목표 위치 이동 실패
   - **해결 방법**: URDF 파일을 통한 정확한 tf 변환 설정, odom 프레임 최적화, 모터 제어 및 IMU 필터링 최적화, `nav2_params.yaml`에서 local & global planner 파라미터 조정을 통한 안정적 이동 구현






---

## ⚙️ 주요 개선 사항
1. **SLAM 최적화**: CPU 성능을 최적화하여 안정적인 SLAM 실행
2. **로컬라이제이션 안정화**: 맵 세부 정보 보완을 통한 위치 추정 정확도 향상
3. **장애물 검출 및 거리 추정**: Depth Camera를 활용하여 장애물과 계단 거리 인식 정확도 개선

---






## 🎞️ 모든 사진과 동영상

### 졸업작품 판넬
![KakaoTalk_20240923_051402601](https://github.com/user-attachments/assets/7107e888-6b60-4894-a8d8-d402c45cea91)

### 개발 초기 모습
![image](https://github.com/user-attachments/assets/01b3ccf4-0242-4eee-8216-6791a8c6ade6)
![image](https://github.com/user-attachments/assets/6171fb87-458b-4974-936e-c0a68f83e1da)
![image](https://github.com/user-attachments/assets/2b3042e2-b87a-4200-a635-66ed4e460557)

![image](https://github.com/user-attachments/assets/c7f928da-629e-4b5e-a22c-dad3343324ee)
![image](https://github.com/user-attachments/assets/32ad4fce-f387-4667-8f54-efa4e67cc4e8)


---

## 📬 연락처

- Email: [your-email@example.com](mailto:your-email@example.com)
- GitHub: [GitHub Profile](https://github.com/yourusername)
- LinkedIn: [LinkedIn Profile](https://linkedin.com/in/yourusername)

```
