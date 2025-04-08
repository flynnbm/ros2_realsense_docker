# ROS 2 Jazzy + RealSense Docker Setup for Raspberry Pi 5

This repository provides a complete setup for running a RealSense D435i camera with ROS 2 Jazzy inside a Docker container on a Raspberry Pi 5. It includes networking via CycloneDDS, and is built for plug-and-play ROS 2 device streaming across machines.

## Docker Image

Pre-built image is hosted on Docker Hub:

```bash
docker pull flynnbm/ros2-jazzy-realsense
```

Contains:

- ROS 2 Jazzy (Ubuntu 24.04)
- RealSense ROS 2 packages (`realsense2_camera`)
- CycloneDDS
- Tools like `rs-enumerate-devices`, `udev`, `nano`
- Prebuilt `/root/ros2_ws` with RealSense launch files

## Requirements

- Docker & Docker Compose installed on Raspberry Pi 5
- RealSense camera (D435i tested)
- USB 2.0 or 3.0 cable (USB 3.0 strongly preferred if functional, USB 3.0 ports on Raspberry Pi occasionally present issues)
- Another machine with ROS 2 Jazzy installed (for remote visualization)

## Setup Instructions

### 1. Installing Docker & Compose on the Pi

```bash
sudo apt update
sudo apt install docker.io docker-compose
```

If `docker compose` doesn't work, use the legacy:

```bash
docker-compose up -d
```

### 2. Clone the repository

```bash
git clone https://github.com/flynnbm/ros2_realsense_docker.git
cd ros2_realsense_docker
```

### 3. Create or edit `cyclonedds.xml`

Replace `<host.machine.ip.address>` and `<raspberry.pi.ip.address>` with real IPs:

```xml
<Peers>
  <Peer address="<host.machine.ip.address>"/>
  <Peer address="<raspberry.pi.ip.address>"/>
</Peers>
```

Place this file in the repository root (next to `docker-compose.yml`).

### 4. Start the container

```bash
docker-compose up -d
```

Then enter the container:

```bash
docker exec -it ros2_jazzy_dev bash
```

### 5. Launch the RealSense node

Inside the container:

```bash
ros2 launch realsense2_camera rs_pointcloud_launch.py
```

### 6. View on main PC

On your ROS 2 Jazzy PC:

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file://$HOME/cyclonedds.xml
```

> Optional: add those lines to your `.bashrc`

## Notes and Tips

- Plug the RealSense camera **into the Pi before starting the container**  
  If not detected, restart the container after plugging it in
- USB 3.0 works best, but some cables or ports may be unreliable — USB 2.0 fallback can still stream
- If topics aren't showing up on your PC, confirm IPs are correct in `cyclonedds.xml`
- Container is configured to source `/root/ros2_ws/install/setup.bash` automatically
- 'ros2-jazzy-realsense` image is prebuilt, so users don’t need to rebuild anything

## Reset Docker Environment (optional)

To fully reset Docker:

```bash
docker ps -q | xargs -r docker stop
docker ps -aq | xargs -r docker rm
docker images -q | xargs -r docker rmi
```

To remove just this image:

```bash
docker stop ros2_jazzy_dev
docker rm ros2_jazzy_dev
docker rmi flynnbm/ros2-jazzy-realsense:latest
```

## Future Improvements

- Auto-launch realsense node on container start
- Add support for other camera/gripper combinations
- Add `ros2 bag record` or logging setup
