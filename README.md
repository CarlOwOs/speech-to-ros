# Voice-Controlled Unitree Go2 Robot

Control your Unitree Go2 quadruped robot through natural voice commands. You can speak to your robot, and it responds both physically and verbally.

## How It Works

The system creates a seamless voice-to-motion pipeline:

1. **Voice Input**: You speak commands into your microphone
2. **Speech Recognition**: OpenAI Whisper transcribes your speech to text
3. **Command Interpretation**: ChatGPT converts the transcript into structured motion commands
4. **Robot Response**: The robot acknowledges your command with a playful verbal response (via ElevenLabs TTS)
5. **Motion Execution**: The quadruped robot executes the movement using ROS 2

## Quick Start

### Prerequisites
1. **Ubuntu 22.04 LTS (Jammy Jellyfish)** - This project requires Ubuntu 22.04. If you're on macOS, consider using VMware Fusion with a [Jammy Jellyfish server image](https://cdimage.ubuntu.com/releases/jammy/release/) and install the GUI with:
   ```bash
   sudo apt install ubuntu-desktop
   ```

2. **Unitree Go2 SDK** - Follow the installation instructions for the unofficial Go2 SDK: [go2_ros2_sdk](https://github.com/abizovnuralem/go2_ros2_sdk)

### Installation

3. **Copy Project Files** - Move the files from this repository to the following locations in your ROS 2 workspace:

   - `paws_driver_node.py` → `ros2_ws/src/go2_robot_sdk/go2_robot_sdk/paws_driver_node.py`
   - `paws_go2_control_node.py` → `ros2_ws/src/go2_robot_sdk/go2_robot_sdk/paws_go2_control_node.py`
   - `test.launch.py` → `ros2_ws/src/go2_robot_sdk/launch/test.launch.py`
   - `setup.py` → `ros2_ws/src/go2_robot_sdk/setup.py`
   - `speech-to-ros.py` → `ros2_ws/src/gpt/speech-to-ros.py`

4. **Configure API Keys** - Add your API keys to the following files:
   - **ElevenLabs API key**: Line 30 in `test.launch.py` (free tier is sufficient)
   - **OpenAI API key**: Line 31 in `speech-to-ros.py`

5. **Build the Workspace**
   ```bash
   source /opt/ros/$ROS_DISTRO/setup.bash
   rosdep install --from-paths src --ignore-src -r -y
   colcon build
   ```
   
   For subsequent builds, use `colcon build --packages-select <package_name>` to rebuild only modified packages.

### Robot Setup

6. **Network Connection**:
   - Connect both your phone and computer to the same WiFi network or hotspot (EPFL WiFi does not work for this)
   - Use the Unitree Go app on your phone to connect to the robot
   - Set the robot to **Classic mode** (provides better responsiveness to motion commands)
   - Note the robot's IP address from the app
   - **Important**: Completely close the app after getting the IP, as the robot can only connect to one device at a time

### Running the System

7. **Launch the Robot SDK**
   ```bash
   source install/setup.bash
   export ROBOT_IP="<your_robot_ip>"
   export CONN_TYPE="webrtc"
   ros2 launch go2_robot_sdk robot.launch.py
   ```

8. **Start Voice Control** (in a separate terminal)
   ```bash
   python /path/to/speech-to-ros.py
   ```

You can now speak commands like "move forward one meter", "turn left", or "go back two steps" and watch your robot respond!

## Working on the Project

### Development Environment

This project uses ROS 2 (Robot Operating System) for communication between components. The main development workflow involves:

1. **Modifying Node Files**: Edit the Python nodes in `ros2_ws/src/go2_robot_sdk/go2_robot_sdk/`
2. **Rebuilding**: Use `colcon build --packages-select go2_robot_sdk` after changes
3. **Testing**: Launch the system and test with voice commands or direct ROS 2 topic publishing

### Key ROS 2 Topics

- `/move` - Linear movement commands (Go2Move messages)
- `/rotate` - Rotation commands (Go2RpyCmd messages)
- `/tts` - Text-to-speech output (String messages)
- `/cmd_vel_out` - Low-level velocity commands (Twist messages)
- `/go2_states` - Robot state information (Go2State messages)

### Testing Commands

You can test robot functions directly using ROS 2 commands:

```bash
# Move forward 1 meter
ros2 topic pub --once /move go2_interfaces/msg/Go2Move "{x: 1.0, y: 0.0}"

# Turn left 90 degrees (π/2 radians)
ros2 topic pub --once /rotate go2_interfaces/msg/Go2RpyCmd "{yaw: 1.5708}"

# Make the robot speak
ros2 topic pub --once /tts std_msgs/msg/String "{data: 'Hello, I am a good dog!'}"
```

### Debugging

- **Check robot connection**: Monitor `/go2_states` topic for incoming data
- **Audio issues**: Verify microphone permissions and sound device configuration
- **API errors**: Check your OpenAI and ElevenLabs API keys and rate limits
- **Movement issues**: Ensure robot is in Classic mode and standing upright

## Python Files Documentation

### `paws_driver_node.py`
The core ROS 2 driver node that manages communication with the Unitree Go2 robot.

- Establishes WebRTC connection with the robot
- Subscribes to motion commands (`/cmd_vel_out`, `/command`) and forwards them to the robot
- Publishes robot state data (`/go2_states`) and IMU data (`/imu`)
- Handles robot command queuing and execution
- Manages connection lifecycle and error handling

**Topics:**
- **Subscribes**: `/cmd_vel_out`, `/command`, `/webrtc_req`, `/stand_down`
- **Publishes**: `/go2_states`, `/imu`

### `paws_go2_control_node.py`
High-level motion control node that converts movement goals into velocity commands.

- Processes movement commands (`/move`) and rotation commands (`/rotate`)
- Implements goal-based navigation with position feedback
- Converts global coordinates to robot-local coordinate system
- Manages movement queues for sequential command execution
- Provides velocity control with acceleration/deceleration profiles

**Topics:**
- **Subscribes**: `/move`, `/rotate`, `/stop`, `/go2_states`, `/imu`
- **Publishes**: `/command`, `/cmd_vel_out`, `/goal_reached`

### `speech-to-ros.py`
Voice interface that bridges human speech with robot commands.

- Records audio input from microphone (4-second clips)
- Transcribes speech using OpenAI Whisper API
- Converts natural language to structured robot commands via ChatGPT
- Generates playful robot responses using OpenAI API
- Publishes movement and TTS commands to appropriate ROS 2 topics
- Supports both voice input (press Enter) and text input

**AI Integration:**
- **Speech-to-Text**: OpenAI Whisper (`gpt-4o-transcribe`)
- **Command Processing**: OpenAI GPT-4o-mini for motion interpretation
- **Response Generation**: OpenAI GPT-4o-mini for playful verbal responses
- **Text-to-Speech**: ElevenLabs API for robot voice output

## Acknowledgements

This project was developed in the [EPFL AI Team](https://epflaiteam.ch/), and builds upon the work from [com304-go2](https://github.com/dyvrl/com304-go2) and [go2_ros2_sdk](https://github.com/abizovnuralem/go2_ros2_sdk).
