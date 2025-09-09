# Multipass, Remmina 설치
```bash
sudo snap install multipass
sudo snap install remmina
```

**Note** 혹시 설치가 안될 경우 저를 불러주세요 (바이오스 세팅을 건드려줘야 합니다.)
만약 인스턴스가 설치되는 중이라면 정상적으로 진행이 되고 있는것이고, 3분 후 Timeed out 이 뜨더라도 괜찮습니다.

# 우분투 22.04 LTS (Jammy Jellyfish) 인스턴스 생성
```bash
cd ~
git clone https://github.com/BranKein/cloud-init-ros2-humble-husky-sim.git
cd cloud-init-ros2-humble-husky-sim
# cloud-init-ros2-humble-go2-sim 폴더에서
multipass launch 22.04 -n husky-sim -c 4 -m 8G -d 50G --cloud-init cloud-init-ros2-humble.yaml --timeout 300
```
은근 시간이 걸릴 수 있습니다.

우분투 어플리케이션에서 multipass 를 찾아 실행해줍니다.

그 후 왼쪽 사이드바에서 Instance 를 클릭하여 생성된 인스턴스의 목록을 확인합니다.

인스턴스의 이름을 클릭하면 터미널이 실행되고, 아래 명령어를 실행해줍니다.

```bash
sudo apt install ros-humble-clearpath-desktop -y
```

만약 위 명령어를 실행했는데 dkpg lock 관련 에러가 뜬다면 그냥 기다려주시기 바랍니다.

설치가 되고 나면 husky sim 을 위한 파일을 하나 추가해야 합니다.
아래 명령어로 폴더를 만들고 해당 폴더로 이동합니다.

```bash
mkdir -p ~/clearpath
cd ~/clearpath
```

그 후 아래 명령어로 robot.yaml 파일을 생성합니다.
```bash
vi robot.yaml
```

이 프로젝트의 robot.yaml 파일의 내용을 복사하여 붙여넣기 해준 후 자장하여 빠져나옵니다.

마지막으로 아래 명령어를 실행해줍니다.
```bash
source /opt/ros/humble/setup.bash
ros2 run clearpath_generator_common generate_bash -s ~/clearpath
```

위 명령어를 실행한 후 ls 명령어로 clearpath 폴더 안에 setup.bash 파일이 생성되었는지 확인합니다.

~/.bashrc 파일을 열어 맨 아래에 아래 내용을 추가해줍니다.
```bash
source ~/clearpath/setup.bash
```

# 패스워드 설정 및 SSH 접속 설정
아래 명령어로 ubuntu 유저의 패스워드를 설정합니다.
```bash
sudo passwd ubuntu
```
비밀번호는 12341234 로 설정합니다.

아래 명령어로 SSH 접속을 허용하도록 sshd_config 파일을 수정합니다.
```bash
sudo vi /etc/ssh/sshd_config
```

파일에서 PasswordAuthentication no 를 찾아서 주석이 들어가있다면 없애주고 no 를 yes 로 바꿔줍니다. (색이 들어와야 합니다)
그 아래쪽에 있는 PbdInteractiveAuthentication no 도 동일하게 yes 로 바꿔줍니다.
그 후 저장을 하여 빠져나온 후 아래 명령어로 ssh 서비스를 재시작합니다.
```bash
sudo systemctl restart ssh
sudo systemctl restart sshd
```

# Ubuntu Desktop, XRDP 설치

여전히 생성한 인스턴스의 터미널에서 아래 명령어를 실행합니다.

시간이 꽤 소요됩니다.

```bash
sudo apt install ubuntu-desktop xrdp -y
```

설치되는 동안 아래 작업을 진행해 주세요!

터미널은 실행해둔채, 상단 Details 버튼을 클릭하여 PRIVATE IP 를 클릭하여 복사해둡니다.

우분투 어플리케이션 에서 remmina 를 찾아 실행해줍니다.
왼쪽 상단의 + 버튼을 클릭하여 새로운 연결을 생성합니다.

* 이름: husky-sim
* Protocol: RDP
* Server: [복사해둔 PRIVATE IP]
* Username: ubuntu
* Password: 12341234

아래로 스크롤하면 Resolution 을 선택할 수 있는데, Custom 을 누른 후 최대한 큰 해상도를 선택해줍니다.

Ubuntu desktop 의 설치가 안되었더라도 우선 한번 Save and Connect 를 눌러줍니다. (설치가 되지 않았다면 "Cannot connect to ~~~ RDP server 라고 뜨고, Close 를 눌러 닫아주세요.)

Remmina 화면에서 husky-sim 이 보일텐데, 이를 우클릭하여 Copy 를 눌러줍니다. 이름 끝에 FTP 를 붙여주고, Protocol 을 SFTP 로 바꿔줍니다.

다시한번 Save and Connect 를 눌러주고 저를 불러주세요. USB 를 하나 드릴겁니다. 연결된 SFTP 창에서 Upload 버튼을 눌러 제가 드린 USB 에 들어있는 ai-gcs-robot.tar.gz 파일과 webrtc-6030-x86_64.tar.gz 파일을 업로드 해줍니다.

다시 인스턴스의 터미널로 돌아와 아래 명령어를 실행합니다.
```bash
cd ~
tar -xvzf ai-gcs-robot.tar.gz
tar -xvzf webrtc-6030-x86_64.tar.gz

cd ai-gcs-robot/cpp/rtc-sender-lib/third_party
chmod +x fetch_webrtc_source.sh
./fetch_webrtc_source.sh 
```
해당 작업은 짧으면 10분, 길면 40분 이상도 걸리는 작업이므로 중간에 절대로 터미널을 끄거나 하지 말아주세요.

# Husky WS 세팅
```bash
ln -s ~/x86_64 ~/ai-gcs-robot/cpp/rtc-sender-lib/third_party/webrtc/src/out/Default

cd ~

mkdir -p ~/husky_ws/src
cd ~/husky_ws/src
ln -s ~/ai-gcs-robot/cpp/examples/husky_ros2 ~/husky_ws/src/husky_rtc_sender_eg

sudo apt update
sudo apt install build-essential cmake pkg-config -y
sudo apt install libjsoncpp-dev libssl-dev libboost-all-dev libopencv-dev libwebsocketpp-dev libcpprest-dev libfmt-dev libprotobuf-dev uuid-dev -y

vi ~/husky_ws/src/husky_rtc_sender_eg/CMakeLists.txt
```
CmakeLists.txt 파일을 열고 조금 아래로 내리면 find_package(Protobuf 3.... 이라는 부분이 있는데,
find_package(Protobuf REQUIRED) 로 바꿔줍니다. (버전 제한을 없앰)
그 후 저장하여 빠져나옵니다.

```bash
vi ~/husky_ws/src/husky_rtc_sender_eg/src/common_impl/utils/ros_image2cv_mat.cpp
```
ros_image2cv_mat.cpp 파일을 열고 include 중 cv_bridge 부분을 아래와 같이 바꿔줍니다.
```cpp
#include <cv_bridge/cv_bridge.h>
```

아래 명령어를 실행하여 빌드를 진행합니다.
```bash
cd ~/husky_ws

source /opt/ros/humble/setup.bash
colcon build
```
빌드에 문제가 있는건지 잘 되는건지 잘 모르겠다면 물어봐주세요!

"Summary: 1 package finished [11.7s]" 과 같은 메시지가 뜨면 빌드가 완료된 것입니다.

# 시뮬레이터 설치 및 실행
이제 husky simulator 를 설치해봅시다.

시뮬레이터는 GUI 를 반드시 사용하기 때문에 remmina 로 접속한 후 터미널을 하나 열어 아래 명령어를 실행합니다.

이전에 remmina 로 접속한 이력이 있으니 husky-sim 을 찾아 더블클릭하여 접속합니다.
(화면이 작다면 husky-sim 을 우클릭하여 Edit 를 눌러 해상도를 Custom 에서 가장 크게 변경해주고, Save and Connect 를 눌러 접속합니다.)

Setup 화면이 나올텐데, 적당히 넘어가주고, ubuntu 24 로의 업데이트가 나온다면 Don't Upgrade 를 눌러주세요. 한글 자판도 필요하지 않습니다.

원격 접속한 화면에서 터미널을 열고 싶을 시 왼쪽 상단 의 Activities 를 클릭한 후 Terminal 을 검색하여 실행합니다.

```bash
sudo apt-get update && sudo apt-get install wget
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt-get update && sudo apt-get install ignition-fortress

sudo apt-get update
sudo apt-get install ros-humble-clearpath-simulator -y

# 보라색 화면이 나오면 엔터 -> esc 눌러주세요 

mkdir ~/clearpath_ws/src -p
source /opt/ros/humble/setup.bash
sudo apt install python3-vcstool
cd ~/clearpath_ws
wget https://raw.githubusercontent.com/clearpathrobotics/clearpath_simulator/humble/dependencies.repos
vcs import src < dependencies.repos
rosdep install -r --from-paths src -i -y
colcon build --symlink-install

ros2 launch clearpath_gz simulation.launch.py
```

처음 실행 시에는 시간이 꽤 걸립니다. gazebo 의 까만 화면이 뜨더라도 기다려주세요.

가지보가 실행되면 우측 Topic 을 "a200_0000/cmd_vel" 로 바꿔주고 하단의 화살표를 누르면서 husky 가 움직이는지 확인합니다.

# RTC Sender 실행
가지보는 켜져있는 상태에서 다시 터미널을 열고 (Activities -> Terminal) 아래 명령어를 실행합니다.

이전에 사용하던 터미널이 띄워질텐데, 왼쪽 상단 + 버튼을 눌러 새 터미널을 열어줍니다. (기존 터미널은 닫지 마세요)
```bash
cd ~/husky_ws
source install/setup.bash
ros2 launch husky_rtc_sender_eg husky_rtc_sender_launch.py user_id:=922fbc34-3a5c-4fcc-974a-1ff3f9e4e9e3 robot_id:=7fd3d5c7-e04c-49d4-bef3-5eb7f4d8907b
```
끝에서 첫번째 또는 두번째 줄에 "Changing state from INITIALIZING to READY_TO_CONNECT" 가 뜨면 저한테 말씀해주시면 감사하겠습니다.