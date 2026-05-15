# ROS2 Fundamentals Tutorial
## A Comprehensive 4-Hour Hands-On Lab Guide

**Target Audience:** MSc Students - Robotics/Autonomous Systems Module  
**Duration:** 4 Hours  
**Hardware Required:** None - Pure software tutorial  
**Software Required:** 
- Ubuntu 22.04 (native or VM)
- ROS2 Humble Hawksbill
- Python 3.10+

---

## Table of Contents

1. [Hour 1: ROS2 Basics - Nodes, Topics, and Communication](#hour-1-ros2-basics---nodes-topics-and-communication)
2. [Hour 2: Services and Parameters](#hour-2-services-and-parameters)
3. [Hour 3: Actions and Advanced Concepts](#hour-3-actions-and-advanced-concepts)
4. [Hour 4: RViz and Complete Integration](#hour-4-rviz-and-complete-integration)

---

# HOUR 1: ROS2 Basics - Nodes, Topics, and Communication

## Learning Objectives
- Understand ROS2 architecture and philosophy
- Master nodes and their lifecycle
- Work with topics and publishers/subscribers
- Create workspaces and packages
- Use colcon build system
- Debug with command-line tools

---

## 1.1 What is ROS2?

### Philosophy and Architecture

**ROS2 (Robot Operating System 2)** is a middleware framework designed for building robot applications. It's not an operating system, but rather a set of software libraries and tools.

### Key Concepts:

**1. Distributed System**
- No central master (unlike ROS1)
- Each node is independent
- Nodes discover each other automatically
- More robust and scalable

**2. Communication Patterns**
```
Topics:     Publisher → [Topic] → Subscribers (many-to-many)
Services:   Client → Request → Server → Response (one-to-one)
Actions:    Client → Goal → Server → Feedback → Result (asynchronous)
Parameters: Runtime configuration values
```

**3. Data Distribution Service (DDS)**
- ROS2 uses DDS for communication
- Provides Quality of Service (QoS) controls
- Supports real-time systems
- Better security features

---

## 1.2 Understanding Nodes

### What is a Node?

A **node** is an executable that uses ROS2 to communicate with other nodes. Think of it as a single-purpose executable in your robot system.

**Examples of nodes:**
- Camera driver node
- Path planning node
- Motor controller node
- Sensor processing node

### Node Characteristics:

1. **Independence**: Each node runs as a separate process
2. **Single Purpose**: Each node should do one thing well
3. **Communication**: Nodes communicate via topics, services, or actions
4. **Lifecycle**: Nodes can be started, stopped, and restarted independently

### Visualizing Node Architecture:

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   Camera    │─────────│  Processor  │─────────│   Display   │
│    Node     │ /image  │    Node     │ /result │    Node     │
└─────────────┘         └─────────────┘         └─────────────┘
```

---

## 1.3 Setting Up ROS2 Environment

### Installation (Ubuntu 22.04)

```bash
# Set locale
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# Setup sources
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install ROS2 Humble
sudo apt update
sudo apt upgrade
sudo apt install ros-humble-desktop
sudo apt install ros-dev-tools
sudo apt install python3-colcon-common-extensions
sudo apt install python3-rosdep

# Initialize rosdep
sudo rosdep init
rosdep update
```

### Environment Setup

```bash
# Source ROS2 (add to ~/.bashrc)
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc

# Verify installation
ros2 --version
# Should output: ros2 doctor version <version>

# Test with demo nodes
ros2 run demo_nodes_cpp talker
# In another terminal:
ros2 run demo_nodes_py listener
```

---

## 1.4 Understanding Workspaces

### What is a Workspace?

A **workspace** is a directory where you develop, build, and install ROS2 packages.

### Workspace Structure:

```
ros2_ws/
├── src/              # Source space (your code goes here)
│   ├── package_1/
│   ├── package_2/
│   └── package_3/
├── build/            # Build space (CMake/colcon artifacts)
├── install/          # Install space (executable binaries)
└── log/              # Log files from builds and tests
```

### Creating a Workspace:

```bash
# Create workspace directory
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws

# Build empty workspace
colcon build

# Output:
# Starting >>> <packages>
# Finished <<< <packages>
# Summary: X packages finished

# Source the workspace overlay
source ~/ros2_ws/install/setup.bash

# Add to .bashrc for automatic sourcing
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

### Understanding Overlays:

```
Underlay: /opt/ros/humble  (ROS2 installation)
    ↓
Overlay: ~/ros2_ws         (Your workspace)
    ↓
Your packages have priority over underlay packages
```

---

## 1.5 Creating Your First Package

### What is a Package?

A **package** is the organizational unit for ROS2 code. It contains:
- Source code (Python or C++)
- Configuration files
- Launch files
- Dependencies declaration

### Package Types:

1. **ament_python**: Pure Python packages
2. **ament_cmake**: C++ packages
3. **ament_cmake_python**: Mixed C++ and Python

### Creating a Python Package:

```bash
cd ~/ros2_ws/src

# Create package with dependencies
ros2 pkg create --build-type ament_python py_fundamentals \
  --dependencies rclpy std_msgs geometry_msgs

# Package structure created:
# py_fundamentals/
# ├── package.xml           # Package metadata and dependencies
# ├── setup.py             # Python package setup
# ├── setup.cfg            # Configuration
# ├── py_fundamentals/     # Python module
# │   └── __init__.py
# ├── resource/
# └── test/
```

### Understanding package.xml:

```xml
<?xml version="1.0"?>
<package format="3">
  <name>py_fundamentals</name>
  <version>0.0.1</version>
  <description>ROS2 fundamentals package</description>
  <maintainer email="you@example.com">Your Name</maintainer>
  <license>Apache-2.0</license>

  <!-- Build tool dependency -->
  <buildtool_depend>ament_python</buildtool_depend>

  <!-- Runtime dependencies -->
  <exec_depend>rclpy</exec_depend>
  <exec_depend>std_msgs</exec_depend>
  <exec_depend>geometry_msgs</exec_depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

---

## 1.6 Understanding Topics

### What is a Topic?

A **topic** is a named bus over which nodes exchange **messages**. Topics implement a publish-subscribe pattern:

- **Publishers** send messages to topics
- **Subscribers** receive messages from topics
- Many-to-many communication (multiple publishers and subscribers)

### Topic Characteristics:

```
Unidirectional:  Publisher → Topic → Subscriber
Asynchronous:    No waiting for responses
Continuous:      Data streams continuously
Anonymous:       Publishers don't know about subscribers
```

### Topic Naming Conventions:

```
Good:  /robot/sensors/temperature
       /cmd_vel
       /camera/image_raw

Bad:   /Temperature  (avoid capitals)
       /data         (too generic)
```

---

## 1.7 Lab Exercise 1: Your First Publisher Node

### Understanding the Code Structure:

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/simple_publisher.py`

```python
#!/usr/bin/env python3
"""
Simple Publisher Node
Demonstrates basic publishing to a topic

Key Concepts:
- Node initialization
- Publisher creation
- Timer callbacks
- Message publishing
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SimplePublisher(Node):
    """
    A simple publisher that sends messages periodically
    
    This node demonstrates:
    1. How to create a node class
    2. How to create a publisher
    3. How to use timers for periodic publishing
    4. How to use the logger
    """
    
    def __init__(self):
        # Initialize the node with a unique name
        # This name is used to identify the node in the ROS2 graph
        super().__init__('simple_publisher')
        
        # Create a publisher
        # Parameters:
        #   - Message type: String (from std_msgs)
        #   - Topic name: 'chatter'
        #   - Queue size: 10 (max messages to buffer)
        self.publisher_ = self.create_publisher(String, 'chatter', 10)
        
        # Create a timer
        # Calls timer_callback every 0.5 seconds
        timer_period = 0.5  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)
        
        # Initialize counter
        self.counter = 0
        
        # Log node startup
        self.get_logger().info('Simple Publisher Node has been started')
        self.get_logger().info(f'Publishing to topic: chatter')
        self.get_logger().info(f'Publishing rate: {1/timer_period} Hz')
    
    def timer_callback(self):
        """
        Timer callback function - called periodically
        
        This method is executed every timer_period seconds.
        It creates and publishes a message.
        """
        # Create a message
        msg = String()
        msg.data = f'Hello ROS2: {self.counter}'
        
        # Publish the message
        self.publisher_.publish(msg)
        
        # Log the published message
        self.get_logger().info(f'Publishing: "{msg.data}"')
        
        # Increment counter
        self.counter += 1

def main(args=None):
    """
    Main function - entry point of the node
    
    Flow:
    1. Initialize ROS2
    2. Create node instance
    3. Spin (keep node running)
    4. Cleanup on shutdown
    """
    # Initialize ROS2 Python client library
    rclpy.init(args=args)
    
    # Create node instance
    node = SimplePublisher()
    
    # Keep the node running until interrupted
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        print('\nShutting down...')
    
    # Cleanup
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Detailed Explanation:

**1. Imports:**
```python
import rclpy                 # ROS2 Python library
from rclpy.node import Node  # Node base class
from std_msgs.msg import String  # String message type
```

**2. Class Inheritance:**
```python
class SimplePublisher(Node):
```
All ROS2 nodes inherit from `Node` base class.

**3. Publisher Creation:**
```python
self.publisher_ = self.create_publisher(String, 'chatter', 10)
```
- `String`: Message type
- `'chatter'`: Topic name
- `10`: Queue size (number of messages to buffer)

**4. Timer Creation:**
```python
self.timer = self.create_timer(timer_period, self.timer_callback)
```
Timers execute callbacks periodically without blocking.

**5. Spinning:**
```python
rclpy.spin(node)
```
Keeps the node active and processes callbacks.

---

## 1.8 Lab Exercise 2: Your First Subscriber Node

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/simple_subscriber.py`

```python
#!/usr/bin/env python3
"""
Simple Subscriber Node
Demonstrates receiving messages from a topic

Key Concepts:
- Subscription creation
- Callback functions
- Message handling
- Statistics tracking
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SimpleSubscriber(Node):
    """
    A simple subscriber that receives and processes messages
    
    This node demonstrates:
    1. How to create a subscriber
    2. How to process received messages
    3. How to track statistics
    4. Best practices for callback functions
    """
    
    def __init__(self):
        # Initialize the node
        super().__init__('simple_subscriber')
        
        # Create a subscription
        # Parameters:
        #   - Message type: String
        #   - Topic name: 'chatter' (must match publisher)
        #   - Callback function: listener_callback
        #   - Queue size: 10
        self.subscription = self.create_subscription(
            String,
            'chatter',
            self.listener_callback,
            10
        )
        
        # Prevent unused variable warning
        self.subscription
        
        # Statistics
        self.message_count = 0
        self.total_length = 0
        
        self.get_logger().info('Simple Subscriber Node has been started')
        self.get_logger().info('Listening on topic: chatter')
    
    def listener_callback(self, msg):
        """
        Callback function - executed when message received
        
        Args:
            msg: The received message (String type)
            
        Note:
            - Keep callbacks FAST
            - Don't block or sleep in callbacks
            - Don't call spin() inside callbacks
        """
        # Extract data from message
        received_data = msg.data
        
        # Update statistics
        self.message_count += 1
        self.total_length += len(received_data)
        avg_length = self.total_length / self.message_count
        
        # Log received message with statistics
        self.get_logger().info(
            f'Received: "{received_data}" | '
            f'Count: {self.message_count} | '
            f'Avg length: {avg_length:.1f} chars'
        )

def main(args=None):
    """
    Main function - entry point
    """
    # Initialize ROS2
    rclpy.init(args=args)
    
    # Create node
    node = SimpleSubscriber()
    
    # Spin
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        print('\nShutting down...')
    
    # Cleanup
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 1.9 Using Colcon Build System

### Understanding Colcon:

**Colcon** (collective construction) is the build system for ROS2. It:
- Builds multiple packages in the correct order
- Handles dependencies automatically
- Supports Python and C++ packages
- Creates install space for executables

### Configuring setup.py:

**File:** `~/ros2_ws/src/py_fundamentals/setup.py`

```python
from setuptools import setup

package_name = 'py_fundamentals'

setup(
    name=package_name,
    version='0.0.1',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='Your Name',
    maintainer_email='your.email@example.com',
    description='ROS2 fundamentals tutorial package',
    license='Apache License 2.0',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            # Format: 'executable_name = package.module:function'
            'simple_publisher = py_fundamentals.simple_publisher:main',
            'simple_subscriber = py_fundamentals.simple_subscriber:main',
        ],
    },
)
```

### Building Your Package:

```bash
# Navigate to workspace root
cd ~/ros2_ws

# Build single package
colcon build --packages-select py_fundamentals

# Build all packages
colcon build

# Build with symbolic links (for Python development)
colcon build --symlink-install

# Build with verbose output
colcon build --event-handlers console_direct+

# Clean build
rm -rf build/ install/ log/
colcon build
```

### Understanding Build Output:

```
Starting >>> py_fundamentals
Finished <<< py_fundamentals [0.50s]

Summary: 1 package finished [0.70s]
```

### Source the Workspace:

```bash
# Source the overlay
source ~/ros2_ws/install/setup.bash

# Verify executables are available
ros2 pkg executables py_fundamentals
# Output:
# py_fundamentals simple_publisher
# py_fundamentals simple_subscriber
```

---

## 1.10 Running and Testing Your Nodes

### Launch Publisher:

**Terminal 1:**
```bash
source ~/ros2_ws/install/setup.bash
ros2 run py_fundamentals simple_publisher
```

**Expected Output:**
```
[INFO] [simple_publisher]: Simple Publisher Node has been started
[INFO] [simple_publisher]: Publishing to topic: chatter
[INFO] [simple_publisher]: Publishing rate: 2.0 Hz
[INFO] [simple_publisher]: Publishing: "Hello ROS2: 0"
[INFO] [simple_publisher]: Publishing: "Hello ROS2: 1"
[INFO] [simple_publisher]: Publishing: "Hello ROS2: 2"
```

### Launch Subscriber:

**Terminal 2:**
```bash
source ~/ros2_ws/install/setup.bash
ros2 run py_fundamentals simple_subscriber
```

**Expected Output:**
```
[INFO] [simple_subscriber]: Simple Subscriber Node has been started
[INFO] [simple_subscriber]: Listening on topic: chatter
[INFO] [simple_subscriber]: Received: "Hello ROS2: 0" | Count: 1 | Avg length: 13.0 chars
[INFO] [simple_subscriber]: Received: "Hello ROS2: 1" | Count: 2 | Avg length: 13.0 chars
```

---

## 1.11 ROS2 Command-Line Tools

### Essential Commands:

**Node Commands:**
```bash
# List all running nodes
ros2 node list
# Output: /simple_publisher
#         /simple_subscriber

# Get detailed info about a node
ros2 node info /simple_publisher
# Shows: Subscribers, Publishers, Services, Actions

# List all executables in a package
ros2 pkg executables py_fundamentals
```

**Topic Commands:**
```bash
# List all active topics
ros2 topic list
# Output: /chatter
#         /parameter_events
#         /rosout

# List topics with type
ros2 topic list -t

# Get topic information
ros2 topic info /chatter
# Shows: Type, Publisher count, Subscription count

# Show topic type
ros2 topic type /chatter
# Output: std_msgs/msg/String

# Echo messages from a topic
ros2 topic echo /chatter

# Show message rate
ros2 topic hz /chatter
# Output: average rate: 2.000

# Show message bandwidth
ros2 topic bw /chatter

# Publish to a topic manually
ros2 topic pub /chatter std_msgs/msg/String "data: 'Hello from command line'"

# Publish once
ros2 topic pub --once /chatter std_msgs/msg/String "data: 'One time message'"

# Publish at specific rate
ros2 topic pub --rate 10 /chatter std_msgs/msg/String "data: 'Fast message'"
```

**Interface Commands:**
```bash
# Show message structure
ros2 interface show std_msgs/msg/String
# Output: string data

# List all message types
ros2 interface list

# Show package interfaces
ros2 interface package std_msgs
```

**General Commands:**
```bash
# Get ROS2 version
ros2 --version

# Doctor - check ROS2 setup
ros2 doctor

# List all packages
ros2 pkg list

# Get package path
ros2 pkg prefix py_fundamentals

# Create new package
ros2 pkg create <package_name>
```

---

## 1.12 Exercise 1: Temperature Publisher/Subscriber System

**Task:** Create a temperature monitoring system with these requirements:

1. **temperature_publisher.py:**
   - Publish random temperature (15-35°C) to `/temperature` topic
   - Use `std_msgs/Float32`
   - Publish at 2 Hz
   - Log each published value

2. **temperature_subscriber.py:**
   - Subscribe to `/temperature`
   - Calculate: min, max, average
   - Warn if temperature > 30°C
   - Log statistics every 10 messages

### Answer - temperature_publisher.py:

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float32
import random

class TemperaturePublisher(Node):
    def __init__(self):
        super().__init__('temperature_publisher')
        
        # Publisher
        self.publisher_ = self.create_publisher(Float32, 'temperature', 10)
        
        # Timer (2 Hz = 0.5 seconds)
        self.timer = self.create_timer(0.5, self.publish_temperature)
        
        self.get_logger().info('Temperature Publisher started (2 Hz)')
    
    def publish_temperature(self):
        # Generate random temperature
        temp = random.uniform(15.0, 35.0)
        
        # Create and publish message
        msg = Float32()
        msg.data = temp
        self.publisher_.publish(msg)
        
        # Log
        self.get_logger().info(f'Published temperature: {temp:.2f}°C')

def main(args=None):
    rclpy.init(args=args)
    node = TemperaturePublisher()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Answer - temperature_subscriber.py:

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float32

class TemperatureSubscriber(Node):
    def __init__(self):
        super().__init__('temperature_subscriber')
        
        # Subscriber
        self.subscription = self.create_subscription(
            Float32,
            'temperature',
            self.temperature_callback,
            10
        )
        
        # Statistics
        self.temperatures = []
        self.count = 0
        
        self.get_logger().info('Temperature Subscriber started')
    
    def temperature_callback(self, msg):
        temp = msg.data
        self.temperatures.append(temp)
        self.count += 1
        
        # Check threshold
        if temp > 30.0:
            self.get_logger().warn(f'HIGH TEMPERATURE: {temp:.2f}°C')
        
        # Log statistics every 10 messages
        if self.count % 10 == 0:
            min_temp = min(self.temperatures)
            max_temp = max(self.temperatures)
            avg_temp = sum(self.temperatures) / len(self.temperatures)
            
            self.get_logger().info('='*50)
            self.get_logger().info(f'Statistics after {self.count} readings:')
            self.get_logger().info(f'  Min: {min_temp:.2f}°C')
            self.get_logger().info(f'  Max: {max_temp:.2f}°C')
            self.get_logger().info(f'  Avg: {avg_temp:.2f}°C')
            self.get_logger().info('='*50)

def main(args=None):
    rclpy.init(args=args)
    node = TemperatureSubscriber()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Update setup.py:

```python
entry_points={
    'console_scripts': [
        'simple_publisher = py_fundamentals.simple_publisher:main',
        'simple_subscriber = py_fundamentals.simple_subscriber:main',
        'temperature_publisher = py_fundamentals.temperature_publisher:main',
        'temperature_subscriber = py_fundamentals.temperature_subscriber:main',
    ],
},
```

### Test:

```bash
# Build
cd ~/ros2_ws
colcon build --packages-select py_fundamentals
source install/setup.bash

# Terminal 1
ros2 run py_fundamentals temperature_publisher

# Terminal 2
ros2 run py_fundamentals temperature_subscriber

# Terminal 3 - Monitor
ros2 topic hz /temperature
ros2 topic echo /temperature
```

---

# HOUR 2: Services and Parameters

## Learning Objectives
- Understand service request/response pattern
- Create service servers and clients
- Use parameters for dynamic configuration
- Implement parameter callbacks
- Create launch files for multi-node systems

---

## 2.1 Understanding Services

### What is a Service?

A **service** provides **request/response** communication:
- **Synchronous**: Client waits for response
- **One-to-one**: Single client talks to single server
- **Transient**: No continuous data stream

### Services vs Topics:

| Feature | Topic | Service |
|---------|-------|---------|
| Communication | Continuous stream | Request/Response |
| Pattern | Many-to-many | One-to-one |
| Timing | Asynchronous | Synchronous |
| Use Case | Sensor data | Configuration, computation |

### Service Workflow:

```
1. Client creates request
2. Client calls service (blocking)
3. Server receives request
4. Server processes request
5. Server sends response
6. Client receives response
```

### Common Service Use Cases:

- Trigger camera capture
- Reset odometry
- Calculate path
- Change robot mode
- Query system state

---

## 2.2 Service Message Types

### Service Definition Structure:

```
# Request fields
---
# Response fields
```

### Example: AddTwoInts.srv

```
int64 a
int64 b
---
int64 sum
```

### Viewing Service Types:

```bash
# List all service types
ros2 interface list | grep srv

# Show service structure
ros2 interface show example_interfaces/srv/AddTwoInts
# Output:
# int64 a
# int64 b
# ---
# int64 sum
```

---

## 2.3 Lab Exercise 3: Service Server

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/add_two_ints_server.py`

```python
#!/usr/bin/env python3
"""
Service Server - Adds two integers

Key Concepts:
- Creating a service
- Handling requests
- Sending responses
- Logging service calls
"""

import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts

class AddTwoIntsServer(Node):
    """
    Service server that adds two integers
    
    This demonstrates:
    1. How to create a service server
    2. How to handle incoming requests
    3. How to send responses
    4. Error handling in services
    """
    
    def __init__(self):
        super().__init__('add_two_ints_server')
        
        # Create service
        # Parameters:
        #   - Service type: AddTwoInts
        #   - Service name: 'add_two_ints'
        #   - Callback function: add_two_ints_callback
        self.srv = self.create_service(
            AddTwoInts,
            'add_two_ints',
            self.add_two_ints_callback
        )
        
        # Statistics
        self.request_count = 0
        self.total_sum = 0
        
        self.get_logger().info('Add Two Ints Service Server is ready')
        self.get_logger().info('Service name: add_two_ints')
        self.get_logger().info('Waiting for requests...')
    
    def add_two_ints_callback(self, request, response):
        """
        Service callback function
        
        Args:
            request: AddTwoInts.Request containing a and b
            response: AddTwoInts.Response to fill with sum
            
        Returns:
            response: Filled response object
            
        Note:
            - This is called when service is invoked
            - Must return the response object
            - Keep processing fast
        """
        # Extract request data
        a = request.a
        b = request.b
        
        # Perform calculation
        result = a + b
        
        # Fill response
        response.sum = result
        
        # Update statistics
        self.request_count += 1
        self.total_sum += result
        avg_result = self.total_sum / self.request_count
        
        # Log the operation
        self.get_logger().info(
            f'Request #{self.request_count}: {a} + {b} = {result}'
        )
        self.get_logger().info(
            f'Statistics: Total requests: {self.request_count}, '
            f'Avg result: {avg_result:.2f}'
        )
        
        return response

def main(args=None):
    rclpy.init(args=args)
    node = AddTwoIntsServer()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        print('\nShutting down server...')
    
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 2.4 Lab Exercise 4: Service Client

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/add_two_ints_client.py`

```python
#!/usr/bin/env python3
"""
Service Client - Calls add two ints service

Key Concepts:
- Creating a service client
- Sending requests
- Waiting for responses
- Error handling
"""

import sys
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts

class AddTwoIntsClient(Node):
    """
    Service client that requests integer addition
    
    This demonstrates:
    1. How to create a service client
    2. How to wait for service availability
    3. How to send requests (sync and async)
    4. How to handle responses
    """
    
    def __init__(self):
        super().__init__('add_two_ints_client')
        
        # Create client
        self.client = self.create_client(AddTwoInts, 'add_two_ints')
        
        # Wait for service to be available
        self.get_logger().info('Waiting for service add_two_ints...')
        while not self.client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Service not available, waiting...')
        
        self.get_logger().info('Service is available!')
    
    def send_request(self, a, b):
        """
        Send request to service
        
        Args:
            a: First integer
            b: Second integer
            
        Returns:
            Future object (for async call)
        """
        # Create request
        request = AddTwoInts.Request()
        request.a = a
        request.b = b
        
        self.get_logger().info(f'Sending request: {a} + {b}')
        
        # Send async request
        # Returns immediately with a Future object
        future = self.client.call_async(request)
        
        return future
    
    def send_request_sync(self, a, b):
        """
        Send synchronous request (blocks until response)
        
        Args:
            a: First integer
            b: Second integer
            
        Returns:
            Response object or None if failed
        """
        request = AddTwoInts.Request()
        request.a = a
        request.b = b
        
        self.get_logger().info(f'Sending sync request: {a} + {b}')
        
        try:
            # This blocks until response received
            response = self.client.call(request)
            return response
        except Exception as e:
            self.get_logger().error(f'Service call failed: {e}')
            return None

def main(args=None):
    rclpy.init(args=args)
    
    # Get command-line arguments
    if len(sys.argv) < 3:
        print('Usage: ros2 run py_fundamentals add_two_ints_client <a> <b>')
        print('Example: ros2 run py_fundamentals add_two_ints_client 5 7')
        return
    
    try:
        a = int(sys.argv[1])
        b = int(sys.argv[2])
    except ValueError:
        print('Error: Arguments must be integers')
        return
    
    # Create client node
    node = AddTwoIntsClient()
    
    # Send request (async)
    future = node.send_request(a, b)
    
    # Wait for result
    rclpy.spin_until_future_complete(node, future)
    
    # Get response
    if future.result() is not None:
        response = future.result()
        node.get_logger().info(
            f'Result: {a} + {b} = {response.sum}'
        )
    else:
        node.get_logger().error('Service call failed')
    
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Update setup.py:

```python
'add_two_ints_server = py_fundamentals.add_two_ints_server:main',
'add_two_ints_client = py_fundamentals.add_two_ints_client:main',
```

### Test the Service:

```bash
# Build
colcon build --packages-select py_fundamentals
source install/setup.bash

# Terminal 1 - Server
ros2 run py_fundamentals add_two_ints_server

# Terminal 2 - Client
ros2 run py_fundamentals add_two_ints_client 10 20
# Output: Result: 10 + 20 = 30

# Terminal 3 - List services
ros2 service list
# Output: /add_two_ints

# Call service from command line
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 15, b: 25}"
# Output: sum: 40
```

---

## 2.5 Understanding Parameters

### What are Parameters?

**Parameters** are configuration values that can be set when starting a node or changed during runtime.

### Parameter Types:

- `bool`: True/False
- `int`: Integer values
- `double`: Floating-point values
- `string`: Text values
- `byte_array`: Array of bytes
- `bool_array`: Array of booleans
- `integer_array`: Array of integers
- `double_array`: Array of floats
- `string_array`: Array of strings

### Why Use Parameters?

1. **Configuration**: Set node behavior without code changes
2. **Flexibility**: Change behavior at runtime
3. **Reusability**: Same node, different configurations
4. **Testing**: Easy to test different configurations

---

## 2.6 Lab Exercise 5: Using Parameters

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/parameter_node.py`

```python
#!/usr/bin/env python3
"""
Parameter Node - Demonstrates parameter usage

Key Concepts:
- Declaring parameters
- Getting parameter values
- Parameter callbacks
- Dynamic reconfiguration
"""

import rclpy
from rclpy.node import Node
from rcl_interfaces.msg import ParameterDescriptor, ParameterType
from rcl_interfaces.msg import SetParametersResult

class ParameterNode(Node):
    """
    Demonstrates comprehensive parameter usage
    
    This node shows:
    1. How to declare parameters with descriptions
    2. How to get parameter values
    3. How to handle parameter changes
    4. Different parameter types
    """
    
    def __init__(self):
        super().__init__('parameter_node')
        
        # Declare parameters with descriptors
        # String parameter
        self.declare_parameter(
            'robot_name',
            'DefaultRobot',
            ParameterDescriptor(
                description='Name of the robot',
                type=ParameterType.PARAMETER_STRING
            )
        )
        
        # Double parameter with range
        self.declare_parameter(
            'max_speed',
            1.0,
            ParameterDescriptor(
                description='Maximum speed in m/s',
                type=ParameterType.PARAMETER_DOUBLE,
                additional_constraints='Must be positive'
            )
        )
        
        # Boolean parameter
        self.declare_parameter(
            'enable_safety',
            True,
            ParameterDescriptor(
                description='Enable safety features',
                type=ParameterType.PARAMETER_BOOL
            )
        )
        
        # Integer parameter
        self.declare_parameter(
            'sensor_rate',
            10,
            ParameterDescriptor(
                description='Sensor update rate in Hz',
                type=ParameterType.PARAMETER_INTEGER
            )
        )
        
        # Array parameters
        self.declare_parameter(
            'waypoints',
            [0.0, 0.0, 1.0, 1.0, 2.0, 0.0],
            ParameterDescriptor(
                description='List of waypoint coordinates (x1,y1,x2,y2,...)',
                type=ParameterType.PARAMETER_DOUBLE_ARRAY
            )
        )
        
        # Add parameter callback
        self.add_on_set_parameters_callback(self.parameter_callback)
        
        # Create timer to periodically print parameters
        self.timer = self.create_timer(2.0, self.print_parameters)
        
        self.get_logger().info('Parameter Node started')
        self.get_logger().info('='*60)
        self.print_parameters()
    
    def print_parameters(self):
        """
        Print current parameter values
        """
        # Get parameter values
        robot_name = self.get_parameter('robot_name').value
        max_speed = self.get_parameter('max_speed').value
        enable_safety = self.get_parameter('enable_safety').value
        sensor_rate = self.get_parameter('sensor_rate').value
        waypoints = self.get_parameter('waypoints').value
        
        # Log parameters
        self.get_logger().info('Current Parameters:')
        self.get_logger().info(f'  Robot Name: {robot_name}')
        self.get_logger().info(f'  Max Speed: {max_speed} m/s')
        self.get_logger().info(f'  Safety Enabled: {enable_safety}')
        self.get_logger().info(f'  Sensor Rate: {sensor_rate} Hz')
        self.get_logger().info(f'  Waypoints: {waypoints}')
        self.get_logger().info('='*60)
    
    def parameter_callback(self, params):
        """
        Called when parameters are changed
        
        Args:
            params: List of changed parameters
            
        Returns:
            SetParametersResult: Success or failure
        """
        result = SetParametersResult()
        result.successful = True
        
        for param in params:
            # Validate max_speed
            if param.name == 'max_speed':
                if param.value <= 0:
                    result.successful = False
                    result.reason = 'max_speed must be positive'
                    self.get_logger().error(result.reason)
                    return result
                elif param.value > 10.0:
                    result.successful = False
                    result.reason = 'max_speed cannot exceed 10.0 m/s'
                    self.get_logger().error(result.reason)
                    return result
            
            # Validate sensor_rate
            if param.name == 'sensor_rate':
                if param.value < 1 or param.value > 100:
                    result.successful = False
                    result.reason = 'sensor_rate must be between 1 and 100 Hz'
                    self.get_logger().error(result.reason)
                    return result
            
            # Log parameter change
            self.get_logger().info(
                f'Parameter changed: {param.name} = {param.value}'
            )
        
        return result

def main(args=None):
    rclpy.init(args=args)
    node = ParameterNode()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        print('\nShutting down...')
    
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Using Parameters:

```bash
# Run with default parameters
ros2 run py_fundamentals parameter_node

# Run with custom parameters
ros2 run py_fundamentals parameter_node --ros-args \
  -p robot_name:=MyRobot \
  -p max_speed:=2.5 \
  -p enable_safety:=false \
  -p sensor_rate:=20

# List node parameters
ros2 param list /parameter_node

# Get specific parameter
ros2 param get /parameter_node robot_name
# Output: String value is: DefaultRobot

# Set parameter at runtime
ros2 param set /parameter_node max_speed 3.0
# Output: Set parameter successful

# Try invalid value (should fail)
ros2 param set /parameter_node max_speed -1.0
# Output: Setting parameter failed: max_speed must be positive

# Describe parameter
ros2 param describe /parameter_node max_speed

# Dump all parameters to file
ros2 param dump /parameter_node > params.yaml

# Load parameters from file
ros2 run py_fundamentals parameter_node --ros-args --params-file params.yaml
```

---

## 2.7 Launch Files

### What are Launch Files?

**Launch files** allow you to start multiple nodes with configurations in one command.

### Benefits:

1. Start multiple nodes simultaneously
2. Set parameters for each node
3. Remap topics
4. Configure namespaces
5. Conditional node launching

### Creating a Launch File:

**File:** `~/ros2_ws/src/py_fundamentals/launch/example_launch.py`

```python
"""
Example Launch File
Demonstrates launching multiple nodes with configurations
"""

from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    """
    Generate launch description with multiple nodes
    
    Returns:
        LaunchDescription object
    """
    return LaunchDescription([
        # Node 1: Publisher
        Node(
            package='py_fundamentals',
            executable='simple_publisher',
            name='my_publisher',
            output='screen',
            parameters=[{
                'use_sim_time': False
            }],
            remappings=[
                # Remap /chatter to /custom_topic
                ('chatter', 'custom_topic')
            ]
        ),
        
        # Node 2: Subscriber  
        Node(
            package='py_fundamentals',
            executable='simple_subscriber',
            name='my_subscriber',
            output='screen',
            remappings=[
                ('chatter', 'custom_topic')
            ]
        ),
        
        # Node 3: Parameter node with custom config
        Node(
            package='py_fundamentals',
            executable='parameter_node',
            name='configured_robot',
            output='screen',
            parameters=[{
                'robot_name': 'LaunchedRobot',
                'max_speed': 5.0,
                'enable_safety': True,
                'sensor_rate': 15
            }]
        )
    ])
```

### Update setup.py for Launch Files:

```python
import os
from glob import glob

setup(
    # ... other fields ...
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        # Include launch files
        (os.path.join('share', package_name, 'launch'),
            glob('launch/*.py')),
    ],
)
```

### Using Launch Files:

```bash
# Build
cd ~/ros2_ws
colcon build --packages-select py_fundamentals
source install/setup.bash

# Launch
ros2 launch py_fundamentals example_launch.py

# List nodes (in another terminal)
ros2 node list
# Output:
# /my_publisher
# /my_subscriber
# /configured_robot
```

---

## 2.8 Exercise 2: Calculator Service

**Task:** Create a comprehensive calculator service system:

### Requirements:

1. **calculator_server.py:**
   - Service: `/calculate`
   - Operations: add, subtract, multiply, divide
   - Custom service type or use string for operation
   - Validate inputs (e.g., no division by zero)

2. **calculator_client.py:**
   - Accept command-line arguments: operation, num1, num2
   - Call calculator service
   - Display result

### Answer - First, understand we'll use existing types:

We'll use `example_interfaces/srv/AddTwoInts` creatively or create requests via string messages.

**calculator_server.py:**

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts
import sys

class CalculatorServer(Node):
    """
    Multi-operation calculator service
    
    Uses AddTwoInts service but interprets based on node parameters
    """
    
    def __init__(self):
        super().__init__('calculator_server')
        
        # Declare operation parameter
        self.declare_parameter('operation', 'add')
        
        # Create service
        self.srv = self.create_service(
            AddTwoInts,
            'calculate',
            self.calculate_callback
        )
        
        self.get_logger().info('Calculator Server ready')
        self.get_logger().info(f'Operation mode: {self.get_parameter("operation").value}')
    
    def calculate_callback(self, request, response):
        """Perform calculation based on operation parameter"""
        a = request.a
        b = request.b
        operation = self.get_parameter('operation').value
        
        if operation == 'add':
            result = a + b
        elif operation == 'subtract':
            result = a - b
        elif operation == 'multiply':
            result = a * b
        elif operation == 'divide':
            if b == 0:
                self.get_logger().error('Division by zero!')
                response.sum = 0
                return response
            result = int(a / b)
        else:
            self.get_logger().error(f'Unknown operation: {operation}')
            response.sum = 0
            return response
        
        response.sum = result
        self.get_logger().info(f'{a} {operation} {b} = {result}')
        
        return response

def main(args=None):
    rclpy.init(args=args)
    node = CalculatorServer()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Testing:

```bash
# Terminal 1 - Addition
ros2 run py_fundamentals calculator_server --ros-args -p operation:=add

# Terminal 2 - Test addition
ros2 service call /calculate example_interfaces/srv/AddTwoInts "{a: 10, b: 5}"
# Result: 15

# Restart server with multiply
# Terminal 1
ros2 run py_fundamentals calculator_server --ros-args -p operation:=multiply

# Terminal 2
ros2 service call /calculate example_interfaces/srv/AddTwoInts "{a: 7, b: 6}"
# Result: 42
```

---

# HOUR 3: Actions and Advanced Concepts

## Learning Objectives
- Understand action servers and clients
- Implement long-running tasks
- Handle feedback and cancellation
- Create complex multi-node systems
- Master launch file techniques

---

## 3.1 Understanding Actions

### What are Actions?

**Actions** are for long-running tasks that need:
- **Goal**: Task to accomplish
- **Feedback**: Progress updates
- **Result**: Final outcome
- **Cancellation**: Ability to stop mid-execution

### Actions vs Services:

| Feature | Service | Action |
|---------|---------|--------|
| Duration | Quick | Long-running |
| Feedback | None | Continuous |
| Cancelable | No | Yes |
| Example | Get sensor reading | Navigate to goal |

### Action Workflow:

```
1. Client sends goal
2. Server accepts/rejects goal
3. Server processes (sends feedback)
4. Client can cancel anytime
5. Server sends result
```

### Action Message Structure:

```
# Goal
goal_field1
goal_field2
---
# Result
result_field1
result_field2
---
# Feedback  
feedback_field1
feedback_field2
```

---

## 3.2 Built-in Action Types

### Common Actions:

```bash
# List action types
ros2 interface list | grep action

# Example: Fibonacci action
ros2 interface show example_interfaces/action/Fibonacci
```

**Fibonacci.action:**
```
# Goal
int32 order
---
# Result
int32[] sequence
---
# Feedback
int32[] partial_sequence
```

---

## 3.3 Lab Exercise 6: Action Server

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/fibonacci_action_server.py`

```python
#!/usr/bin/env python3
"""
Fibonacci Action Server
Demonstrates action server implementation

Key Concepts:
- Creating action servers
- Handling goals
- Publishing feedback
- Sending results
- Goal cancellation
"""

import time
import rclpy
from rclpy.action import ActionServer, CancelResponse, GoalResponse
from rclpy.node import Node
from example_interfaces.action import Fibonacci

class FibonacciActionServer(Node):
    """
    Action server that computes Fibonacci sequence
    
    This demonstrates:
    1. Action server creation
    2. Goal handling (accept/reject)
    3. Feedback publishing during execution
    4. Result sending upon completion
    5. Goal cancellation handling
    """
    
    def __init__(self):
        super().__init__('fibonacci_action_server')
        
        # Create action server
        self._action_server = ActionServer(
            self,
            Fibonacci,
            'fibonacci',
            execute_callback=self.execute_callback,
            goal_callback=self.goal_callback,
            cancel_callback=self.cancel_callback
        )
        
        self.get_logger().info('Fibonacci Action Server started')
        self.get_logger().info('Waiting for goals...')
    
    def goal_callback(self, goal_request):
        """
        Called when a new goal is received
        
        Args:
            goal_request: The goal request
            
        Returns:
            GoalResponse.ACCEPT or GoalResponse.REJECT
        """
        order = goal_request.order
        
        # Validate goal
        if order < 0:
            self.get_logger().warn(f'Rejecting goal: order must be >= 0, got {order}')
            return GoalResponse.REJECT
        
        if order > 50:
            self.get_logger().warn(f'Rejecting goal: order too large ({order}), max is 50')
            return GoalResponse.REJECT
        
        self.get_logger().info(f'Accepting goal: Fibonacci({order})')
        return GoalResponse.ACCEPT
    
    def cancel_callback(self, goal_handle):
        """
        Called when goal cancellation is requested
        
        Args:
            goal_handle: Handle to the goal
            
        Returns:
            CancelResponse.ACCEPT or CancelResponse.REJECT
        """
        self.get_logger().info('Received cancel request')
        return CancelResponse.ACCEPT
    
    def execute_callback(self, goal_handle):
        """
        Execute the goal
        
        Args:
            goal_handle: Handle to the goal
            
        Returns:
            Result object
        """
        self.get_logger().info('Executing goal...')
        
        # Get goal
        order = goal_handle.request.order
        
        # Initialize feedback
        feedback_msg = Fibonacci.Feedback()
        feedback_msg.partial_sequence = [0, 1]
        
        # Compute Fibonacci sequence
        for i in range(1, order):
            # Check if goal was canceled
            if goal_handle.is_cancel_requested:
                goal_handle.canceled()
                self.get_logger().info('Goal canceled')
                
                result = Fibonacci.Result()
                result.sequence = feedback_msg.partial_sequence
                return result
            
            # Compute next number
            next_num = feedback_msg.partial_sequence[i] + feedback_msg.partial_sequence[i-1]
            feedback_msg.partial_sequence.append(next_num)
            
            # Publish feedback
            goal_handle.publish_feedback(feedback_msg)
            self.get_logger().info(f'Feedback: {feedback_msg.partial_sequence}')
            
            # Simulate computation time
            time.sleep(0.5)
        
        # Mark goal as succeeded
        goal_handle.succeed()
        
        # Create result
        result = Fibonacci.Result()
        result.sequence = feedback_msg.partial_sequence
        
        self.get_logger().info(f'Goal succeeded! Result: {result.sequence}')
        
        return result

def main(args=None):
    rclpy.init(args=args)
    node = FibonacciActionServer()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        print('\nShutting down...')
    
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 3.4 Lab Exercise 7: Action Client

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/fibonacci_action_client.py`

```python
#!/usr/bin/env python3
"""
Fibonacci Action Client
Demonstrates action client implementation

Key Concepts:
- Creating action clients
- Sending goals
- Receiving feedback
- Getting results
- Canceling goals
"""

import sys
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from example_interfaces.action import Fibonacci

class FibonacciActionClient(Node):
    """
    Action client for Fibonacci sequence computation
    
    This demonstrates:
    1. Action client creation
    2. Goal sending
    3. Feedback handling
    4. Result retrieval
    5. Goal cancellation
    """
    
    def __init__(self):
        super().__init__('fibonacci_action_client')
        
        # Create action client
        self._action_client = ActionClient(
            self,
            Fibonacci,
            'fibonacci'
        )
        
        self.get_logger().info('Fibonacci Action Client started')
    
    def send_goal(self, order):
        """
        Send goal to action server
        
        Args:
            order: Fibonacci order to compute
        """
        # Wait for server
        self.get_logger().info('Waiting for action server...')
        self._action_client.wait_for_server()
        
        # Create goal
        goal_msg = Fibonacci.Goal()
        goal_msg.order = order
        
        self.get_logger().info(f'Sending goal: Fibonacci({order})')
        
        # Send goal asynchronously
        self._send_goal_future = self._action_client.send_goal_async(
            goal_msg,
            feedback_callback=self.feedback_callback
        )
        
        # Register callback for when goal is accepted
        self._send_goal_future.add_done_callback(self.goal_response_callback)
    
    def goal_response_callback(self, future):
        """
        Called when server responds to goal request
        
        Args:
            future: Future object containing goal handle
        """
        goal_handle = future.result()
        
        if not goal_handle.accepted:
            self.get_logger().error('Goal rejected by server')
            return
        
        self.get_logger().info('Goal accepted by server')
        
        # Get result
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)
    
    def feedback_callback(self, feedback_msg):
        """
        Called when feedback is received
        
        Args:
            feedback_msg: Feedback message
        """
        feedback = feedback_msg.feedback
        self.get_logger().info(
            f'Received feedback: {feedback.partial_sequence}'
        )
    
    def get_result_callback(self, future):
        """
        Called when result is received
        
        Args:
            future: Future object containing result
        """
        result = future.result().result
        status = future.result().status
        
        if status == 4:  # SUCCEEDED
            self.get_logger().info(f'Goal succeeded!')
            self.get_logger().info(f'Result: {result.sequence}')
        elif status == 5:  # CANCELED
            self.get_logger().warn('Goal was canceled')
            self.get_logger().info(f'Partial result: {result.sequence}')
        else:
            self.get_logger().error(f'Goal failed with status: {status}')
        
        # Shutdown
        rclpy.shutdown()

def main(args=None):
    rclpy.init(args=args)
    
    # Get order from command line
    if len(sys.argv) < 2:
        print('Usage: ros2 run py_fundamentals fibonacci_action_client <order>')
        print('Example: ros2 run py_fundamentals fibonacci_action_client 10')
        return
    
    try:
        order = int(sys.argv[1])
    except ValueError:
        print('Error: order must be an integer')
        return
    
    # Create client and send goal
    node = FibonacciActionClient()
    node.send_goal(order)
    
    # Spin
    rclpy.spin(node)

if __name__ == '__main__':
    main()
```

### Update setup.py:

```python
'fibonacci_action_server = py_fundamentals.fibonacci_action_server:main',
'fibonacci_action_client = py_fundamentals.fibonacci_action_client:main',
```

### Test Actions:

```bash
# Build
cd ~/ros2_ws
colcon build --packages-select py_fundamentals
source install/setup.bash

# Terminal 1 - Server
ros2 run py_fundamentals fibonacci_action_server

# Terminal 2 - Client
ros2 run py_fundamentals fibonacci_action_client 10

# Terminal 3 - Monitor action
ros2 action list
ros2 action info /fibonacci

# Send goal from command line
ros2 action send_goal /fibonacci example_interfaces/action/Fibonacci "{order: 5}"

# Send goal and show feedback
ros2 action send_goal /fibonacci example_interfaces/action/Fibonacci "{order: 10}" --feedback
```

---

## 3.5 Exercise 3: Countdown Timer Action

**Task:** Create a countdown timer action

### Requirements:

1. **countdown_action_server.py:**
   - Action goal: duration in seconds
   - Feedback: remaining time
   - Result: success message
   - Allow cancellation

2. **countdown_action_client.py:**
   - Send duration goal
   - Display feedback
   - Show result

### Answer - Create custom action:

**File:** `~/ros2_ws/src/py_fundamentals/action/Countdown.action`

```
# Goal
int32 duration
---
# Result
string message
bool success
---
# Feedback
int32 remaining_time
```

### Update package.xml:

```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

### Update CMakeLists.txt (if C++) or for Python, we'll use existing types creatively.

For simplicity in this tutorial, let's use Fibonacci action structure repurposed:

**countdown_action_server.py:**

```python
#!/usr/bin/env python3
import time
import rclpy
from rclpy.action import ActionServer
from rclpy.node import Node
from example_interfaces.action import Fibonacci

class CountdownServer(Node):
    def __init__(self):
        super().__init__('countdown_server')
        
        self._action_server = ActionServer(
            self,
            Fibonacci,
            'countdown',
            self.execute_callback
        )
        
        self.get_logger().info('Countdown Server ready')
    
    def execute_callback(self, goal_handle):
        self.get_logger().info(f'Starting countdown from {goal_handle.request.order}')
        
        duration = goal_handle.request.order
        feedback_msg = Fibonacci.Feedback()
        
        for i in range(duration, 0, -1):
            if goal_handle.is_cancel_requested:
                goal_handle.canceled()
                self.get_logger().info('Countdown canceled')
                result = Fibonacci.Result()
                result.sequence = [i]
                return result
            
            feedback_msg.partial_sequence = [i]
            goal_handle.publish_feedback(feedback_msg)
            self.get_logger().info(f'Countdown: {i}')
            time.sleep(1.0)
        
        goal_handle.succeed()
        result = Fibonacci.Result()
        result.sequence = [0]  # Done
        self.get_logger().info('Countdown complete!')
        
        return result

def main(args=None):
    rclpy.init(args=args)
    node = CountdownServer()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

# HOUR 4: RViz and Complete Integration

## Learning Objectives
- Master RViz visualization tool
- Visualize nodes, topics, and tf transforms
- Create markers and interactive visualizations
- Build complete multi-node system
- Implement final integration project

---

## 4.1 Understanding RViz

### What is RViz?

**RViz** (ROS Visualization) is a 3D visualization tool for ROS2 that helps you:
- Visualize sensor data
- Debug robot behavior
- Display tf transforms
- Show custom markers
- Monitor topic data visually

### Starting RViz:

```bash
# Launch RViz
rviz2

# Launch with config file
rviz2 -d my_config.rviz
```

---

## 4.2 RViz Basic Configuration

### Interface Overview:

```
┌─────────────────────────────────────────────┐
│ Menu Bar                                     │
├───────────┬─────────────────────────────────┤
│  Displays │  3D Viewport                    │
│  Panel    │                                 │
│           │  - Grid                         │
│  + TF     │  - Axes                         │
│  + Grid   │  - Robot Model                  │
│  + Marker │  - Interactive                  │
│           │                                 │
├───────────┴─────────────────────────────────┤
│ Tool Properties / Views Panel               │
└─────────────────────────────────────────────┘
```

### Key Components:

1. **Fixed Frame**: Reference coordinate frame
2. **Displays Panel**: Add/configure visualizations
3. **3D Viewport**: Main visualization area
4. **Tool Properties**: Interaction settings
5. **Views Panel**: Camera controls

---

## 4.3 Lab Exercise 8: Publishing Markers to RViz

**File:** `~/ros2_ws/src/py_fundamentals/py_fundamentals/marker_publisher.py`

```python
#!/usr/bin/env python3
"""
Marker Publisher
Publishes visualization markers to RViz

Key Concepts:
- Creating visualization markers
- Different marker types
- Marker properties
- Animation in RViz
"""

import rclpy
from rclpy.node import Node
from visualization_msgs.msg import Marker
from geometry_msgs.msg import Point
import math

class MarkerPublisher(Node):
    """
    Publishes various markers for RViz visualization
    
    Marker Types:
    - SPHERE: 3D sphere
    - CUBE: 3D box
    - CYLINDER: 3D cylinder
    - ARROW: Directional arrow
    - LINE_STRIP: Connected line segments
    - POINTS: Point cloud
    - TEXT_VIEW_FACING: 2D text
    """
    
    def __init__(self):
        super().__init__('marker_publisher')
        
        # Publisher
        self.marker_pub = self.create_publisher(
            Marker,
            'visualization_marker',
            10
        )
        
        # Timer
        self.timer = self.create_timer(0.1, self.publish_markers)
        
        # Animation counter
        self.count = 0
        
        self.get_logger().info('Marker Publisher started')
        self.get_logger().info('Open RViz and add Marker display on /visualization_marker')
    
    def publish_markers(self):
        """Publish different types of markers"""
        self.count += 1
        
        # Publish different markers
        self.publish_sphere()
        self.publish_cube()
        self.publish_arrow()
        self.publish_line_strip()
        self.publish_text()
    
    def publish_sphere(self):
        """Publish animated sphere"""
        marker = Marker()
        
        # Header
        marker.header.frame_id = "map"
        marker.header.stamp = self.get_clock().now().to_msg()
        
        # Marker properties
        marker.ns = "shapes"
        marker.id = 0
        marker.type = Marker.SPHERE
        marker.action = Marker.ADD
        
        # Position (animated)
        marker.pose.position.x = math.sin(self.count * 0.1) * 2.0
        marker.pose.position.y = math.cos(self.count * 0.1) * 2.0
        marker.pose.position.z = 1.0
        
        # Orientation
        marker.pose.orientation.w = 1.0
        
        # Scale
        marker.scale.x = 0.5
        marker.scale.y = 0.5
        marker.scale.z = 0.5
        
        # Color (RGBA)
        marker.color.r = 1.0
        marker.color.g = 0.0
        marker.color.b = 0.0
        marker.color.a = 1.0
        
        # Lifetime (0 = forever)
        marker.lifetime.sec = 0
        
        self.marker_pub.publish(marker)
    
    def publish_cube(self):
        """Publish rotating cube"""
        marker = Marker()
        
        marker.header.frame_id = "map"
        marker.header.stamp = self.get_clock().now().to_msg()
        
        marker.ns = "shapes"
        marker.id = 1
        marker.type = Marker.CUBE
        marker.action = Marker.ADD
        
        # Position
        marker.pose.position.x = 0.0
        marker.pose.position.y = 0.0
        marker.pose.position.z = 1.0
        
        # Orientation (rotating)
        angle = self.count * 0.05
        marker.pose.orientation.z = math.sin(angle / 2)
        marker.pose.orientation.w = math.cos(angle / 2)
        
        # Scale
        marker.scale.x = 1.0
        marker.scale.y = 1.0
        marker.scale.z = 1.0
        
        # Color (Green)
        marker.color.r = 0.0
        marker.color.g = 1.0
        marker.color.b = 0.0
        marker.color.a = 0.8
        
        self.marker_pub.publish(marker)
    
    def publish_arrow(self):
        """Publish arrow pointing at sphere"""
        marker = Marker()
        
        marker.header.frame_id = "map"
        marker.header.stamp = self.get_clock().now().to_msg()
        
        marker.ns = "arrows"
        marker.id = 2
        marker.type = Marker.ARROW
        marker.action = Marker.ADD
        
        # Start and end points
        start = Point()
        start.x = 0.0
        start.y = 0.0
        start.z = 0.5
        
        end = Point()
        end.x = math.sin(self.count * 0.1) * 2.0
        end.y = math.cos(self.count * 0.1) * 2.0
        end.z = 1.0
        
        marker.points = [start, end]
        
        # Scale
        marker.scale.x = 0.1  # Shaft diameter
        marker.scale.y = 0.2  # Head diameter
        marker.scale.z = 0.2  # Head length
        
        # Color (Blue)
        marker.color.r = 0.0
        marker.color.g = 0.0
        marker.color.b = 1.0
        marker.color.a = 1.0
        
        self.marker_pub.publish(marker)
    
    def publish_line_strip(self):
        """Publish trail line"""
        marker = Marker()
        
        marker.header.frame_id = "map"
        marker.header.stamp = self.get_clock().now().to_msg()
        
        marker.ns = "trails"
        marker.id = 3
        marker.type = Marker.LINE_STRIP
        marker.action = Marker.ADD
        
        # Create spiral points
        points = []
        for i in range(100):
            p = Point()
            angle = i * 0.1
            radius = i * 0.02
            p.x = math.cos(angle) * radius
            p.y = math.sin(angle) * radius
            p.z = 0.0
            points.append(p)
        
        marker.points = points
        
        # Scale (line width)
        marker.scale.x = 0.05
        
        # Color (Yellow)
        marker.color.r = 1.0
        marker.color.g = 1.0
        marker.color.b = 0.0
        marker.color.a = 1.0
        
        self.marker_pub.publish(marker)
    
    def publish_text(self):
        """Publish text marker"""
        marker = Marker()
        
        marker.header.frame_id = "map"
        marker.header.stamp = self.get_clock().now().to_msg()
        
        marker.ns = "text"
        marker.id = 4
        marker.type = Marker.TEXT_VIEW_FACING
        marker.action = Marker.ADD
        
        # Position
        marker.pose.position.x = 0.0
        marker.pose.position.y = 0.0
        marker.pose.position.z = 2.0
        marker.pose.orientation.w = 1.0
        
        # Text
        marker.text = f"Count: {self.count}"
        
        # Scale (text height)
        marker.scale.z = 0.3
        
        # Color (White)
        marker.color.r = 1.0
        marker.color.g = 1.0
        marker.color.b = 1.0
        marker.color.a = 1.0
        
        self.marker_pub.publish(marker)

def main(args=None):
    rclpy.init(args=args)
    node = MarkerPublisher()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        print('\nShutting down...')
    
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Visualizing in RViz:

```bash
# Terminal 1 - Run marker publisher
ros2 run py_fundamentals marker_publisher

# Terminal 2 - Launch RViz
rviz2

# In RViz:
# 1. Set Fixed Frame to "map"
# 2. Add -> Marker
# 3. Topic: /visualization_marker
# 4. Watch the animated visualization!
```

---

## 4.4 Final Integration Project: Robot State Publisher

**Task:** Create a complete system that:
1. Publishes robot "pose" (position + orientation)
2. Visualizes robot path in RViz
3. Allows control via parameters
4. Logs all data
5. Uses services to reset/control robot

### Project Files:

**robot_controller.py:**

```python
#!/usr/bin/env python3
"""
Robot Controller - Final Integration Project
Demonstrates all ROS2 concepts learned

Features:
- Parameter-based configuration
- Topic publishing (pose, path)
- Service server (reset, set_speed)
- Action server (move_to_goal)
- RViz visualization
"""

import rclpy
from rclpy.node import Node
from rclpy.action import ActionServer
from geometry_msgs.msg import PoseStamped, Point
from nav_msgs.msg import Path
from std_srvs.srv import Trigger
from example_interfaces.srv import SetBool
from example_interfaces.action import Fibonacci
from visualization_msgs.msg import Marker
import math

class RobotController(Node):
    def __init__(self):
        super().__init__('robot_controller')
        
        # Parameters
        self.declare_parameter('robot_name', 'MyRobot')
        self.declare_parameter('update_rate', 10.0)
        self.declare_parameter('max_speed', 1.0)
        
        # State
        self.x = 0.0
        self.y = 0.0
        self.theta = 0.0
        self.enabled = True
        
        # Publishers
        self.pose_pub = self.create_publisher(PoseStamped, 'robot_pose', 10)
        self.path_pub = self.create_publisher(Path, 'robot_path', 10)
        self.marker_pub = self.create_publisher(Marker, 'robot_marker', 10)
        
        # Services
        self.reset_srv = self.create_service(Trigger, 'reset_pose', self.reset_callback)
        self.enable_srv = self.create_service(SetBool, 'enable_robot', self.enable_callback)
        
        # Action server
        self._action_server = ActionServer(
            self,
            Fibonacci,
            'move_robot',
            self.move_callback
        )
        
        # Timer
        rate = self.get_parameter('update_rate').value
        self.timer = self.create_timer(1.0/rate, self.update)
        
        # Path history
        self.path = Path()
        self.path.header.frame_id = "map"
        
        self.get_logger().info('='*60)
        self.get_logger().info('Robot Controller Started')
        self.get_logger().info(f'Robot Name: {self.get_parameter("robot_name").value}')
        self.get_logger().info(f'Update Rate: {rate} Hz')
        self.get_logger().info('='*60)
    
    def update(self):
        """Update robot state and publish"""
        if not self.enabled:
            return
        
        # Simple circular motion
        speed = self.get_parameter('max_speed').value
        dt = 1.0 / self.get_parameter('update_rate').value
        
        self.theta += 0.5 * dt
        self.x = 2.0 * math.cos(self.theta)
        self.y = 2.0 * math.sin(self.theta)
        
        # Publish pose
        pose_msg = PoseStamped()
        pose_msg.header.stamp = self.get_clock().now().to_msg()
        pose_msg.header.frame_id = "map"
        pose_msg.pose.position.x = self.x
        pose_msg.pose.position.y = self.y
        pose_msg.pose.position.z = 0.0
        
        # Orientation from theta
        pose_msg.pose.orientation.z = math.sin(self.theta / 2)
        pose_msg.pose.orientation.w = math.cos(self.theta / 2)
        
        self.pose_pub.publish(pose_msg)
        
        # Update path
        self.path.poses.append(pose_msg)
        if len(self.path.poses) > 1000:  # Limit path length
            self.path.poses.pop(0)
        
        self.path.header.stamp = self.get_clock().now().to_msg()
        self.path_pub.publish(self.path)
        
        # Publish marker
        self.publish_robot_marker()
    
    def publish_robot_marker(self):
        """Publish robot visualization marker"""
        marker = Marker()
        marker.header.frame_id = "map"
        marker.header.stamp = self.get_clock().now().to_msg()
        marker.ns = "robot"
        marker.id = 0
        marker.type = Marker.ARROW
        marker.action = Marker.ADD
        
        marker.pose.position.x = self.x
        marker.pose.position.y = self.y
        marker.pose.position.z = 0.5
        marker.pose.orientation.z = math.sin(self.theta / 2)
        marker.pose.orientation.w = math.cos(self.theta / 2)
        
        marker.scale.x = 0.5
        marker.scale.y = 0.1
        marker.scale.z = 0.1
        
        marker.color.r = 1.0
        marker.color.g = 0.5
        marker.color.b = 0.0
        marker.color.a = 1.0
        
        self.marker_pub.publish(marker)
    
    def reset_callback(self, request, response):
        """Reset robot pose"""
        self.x = 0.0
        self.y = 0.0
        self.theta = 0.0
        self.path.poses.clear()
        
        response.success = True
        response.message = "Robot pose reset"
        self.get_logger().info("Reset robot pose")
        
        return response
    
    def enable_callback(self, request, response):
        """Enable/disable robot"""
        self.enabled = request.data
        
        response.success = True
        response.message = f"Robot {'enabled' if self.enabled else 'disabled'}"
        self.get_logger().info(response.message)
        
        return response
    
    def move_callback(self, goal_handle):
        """Move robot action"""
        self.get_logger().info('Executing move action...')
        
        # Simulate movement
        feedback_msg = Fibonacci.Feedback()
        feedback_msg.partial_sequence = [0]
        
        for i in range(goal_handle.request.order):
            goal_handle.publish_feedback(feedback_msg)
            feedback_msg.partial_sequence.append(i)
        
        goal_handle.succeed()
        
        result = Fibonacci.Result()
        result.sequence = feedback_msg.partial_sequence
        return result

def main(args=None):
    rclpy.init(args=args)
    node = RobotController()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### Complete Launch File:

**File:** `~/ros2_ws/src/py_fundamentals/launch/robot_system_launch.py`

```python
"""
Complete Robot System Launch
Launches all nodes with RViz visualization
"""

from launch import LaunchDescription
from launch_ros.actions import Node
from launch.actions import ExecuteProcess

def generate_launch_description():
    return LaunchDescription([
        # Robot Controller
        Node(
            package='py_fundamentals',
            executable='robot_controller',
            name='robot_controller',
            output='screen',
            parameters=[{
                'robot_name': 'IntegratedRobot',
                'update_rate': 20.0,
                'max_speed': 1.5
            }]
        ),
        
        # RViz
        Node(
            package='rviz2',
            executable='rviz2',
            name='rviz2',
            output='screen'
        ),
        
        # Optional: Data logger
        ExecuteProcess(
            cmd=['ros2', 'bag', 'record', '-a'],
            output='screen'
        )
    ])
```

### Testing the Complete System:

```bash
# Build
cd ~/ros2_ws
colcon build --packages-select py_fundamentals
source install/setup.bash

# Launch complete system
ros2 launch py_fundamentals robot_system_launch.py

# In RViz:
# 1. Fixed Frame: "map"
# 2. Add -> Path (Topic: /robot_path)
# 3. Add -> PoseStamped (Topic: /robot_pose)
# 4. Add -> Marker (Topic: /robot_marker)

# In another terminal - Test services
ros2 service call /reset_pose std_srvs/srv/Trigger

ros2 service call /enable_robot example_interfaces/srv/SetBool "{data: false}"
ros2 service call /enable_robot example_interfaces/srv/SetBool "{data: true}"

# Monitor system
ros2 node list
ros2 topic list
ros2 service list
ros2 param list
```

---

## Summary and Key Takeaways

### Hour 1: Fundamentals
✅ ROS2 architecture and philosophy  
✅ Nodes and their lifecycle  
✅ Topics and pub/sub pattern  
✅ Workspaces and packages  
✅ Colcon build system  
✅ Command-line tools

### Hour 2: Services and Parameters
✅ Service request/response pattern  
✅ Creating servers and clients  
✅ Parameter configuration  
✅ Dynamic reconfiguration  
✅ Launch files

### Hour 3: Actions
✅ Action servers and clients  
✅ Long-running tasks  
✅ Feedback and cancellation  
✅ Goal management  
✅ Complex workflows

### Hour 4: Visualization and Integration
✅ RViz configuration  
✅ Marker visualization  
✅ Complete system integration  
✅ Multi-node coordination  
✅ Production-ready systems

---

## Additional Resources

### Official Documentation:
- ROS2 Documentation: https://docs.ros.org/en/humble/
- ROS2 Tutorials: https://docs.ros.org/en/humble/Tutorials.html
- ROS2 Design: https://design.ros2.org/

### Community Resources:
- ROS Discourse: https://discourse.ros.org/
- ROS Answers: https://answers.ros.org/
- GitHub: https://github.com/ros2

### Next Steps:
1. Learn tf2 (coordinate transformations)
2. Explore Nav2 (navigation stack)
3. Study robot simulation (Gazebo)
4. Implement SLAM algorithms
5. Add computer vision (OpenCV)
6. Deploy on real robots

---

**End of Tutorial**
