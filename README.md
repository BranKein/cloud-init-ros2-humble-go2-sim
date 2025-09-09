# Multipass, Remmina 설치
```bash
sudo snap install multipass
sudo snap install remmina
```

**Note** 혹시 설치가 안될 경우 저를 불러주세요 (바이오스 세팅을 건드려줘야 합니다.)
만약 인스턴스가 설치되는 중이라면 정상적으로 진행이 되고 있는것이고, 3분 후 Timeed out 이 뜨더라도 괜찮습니다.

# 우분투 22.04 LTS (Jammy Jellyfish) 인스턴스 생성
```bash
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

# 패스워드 설정
아래 명령어로 ubuntu 유저의 패스워드를 설정합니다.
```bash
sudo passwd ubuntu
```
비밀번호는 12341234 로 설정합니다.

# Ubuntu Desktop, XRDP 설치

여전히 생성한 인스턴스의 터미널에서 아래 명령어를 실행합니다.

시간이 꽤 소요됩니다.

```bash
sudo apt install ubuntu-desktop xrdp -y
```

설치가 완료되면 우분투 어플리케이션 에서 remmina 를 찾아 실행해줍니다.
왼쪽 상단의 + 버튼을 클릭하여 새로운 연결을 생성합니다.

이름: husky-sim
Protocol: RDP
Server: <multipass list 명령어로 확인한 인스턴스의 IP 주소>
Username: ubuntu