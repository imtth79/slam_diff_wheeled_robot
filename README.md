# slam_diff_wheeled_robot
* Nhớ chạy roscore khi bắt đầu mở command line và mở thêm một task terminal rồi chạy. Và cũng đừng quên đóng roscore khi đã chạy xong hết bằng tổ hợp phím: Ctrl + C*
## 1. Chuẩn bị:
   + Cài ROS kinetic và Gazebo 7.0 (recommend)
   + Để có thể mô phỏng robot 3D với ROS cần tải 2 package sau: urdf và xacro (tutorial và cách tải có trên ROS Wiki, tải bằng sudo apt-get install ros-<distro>-<package> là được)
  +  Để trực quan hóa (visualization) robot 3D và các kết quả chạy của robot 3D ngoài gazebo còn cần tải thêm rviz nữa.
  +  Để mô phỏng robot bằng Gazebo và ROS thì cần tải cả các package sau: 
     > gazebo_ros_pkgs: Chứa các gói và tools để làm giao diện trên Gazebo từ ROS.
     > gazebo-msgs: Chứa các messages và service data structures để làm giao diện trên Gazebo từ ROS.
     > gazebo-plugins: Chứa các Gazebo plugins dùng cho sensors, actuators, v.v..
     > gazebo-ros-control: Chứa các standard controllers để giao tiếp giữa ROS và Gazebo.
  +  Để dùng cho slam, navigation (localization) và map-server cần tải nốt navigation-stack package cho ROS kinetic là xong phần chuẩn bị.
  
## 2. Bài toán chính: Building a map using SLAM 
   + Tạo catkin_ws theo hướng dẫn trên ROS Tutorial.
   + Điều hướng: cd catkin_ws/src và clone các package ở repo này.
   + cd .. và sau đó thực hiện make file: catkin_make (file CMakeList sẽ được tạo tự động sau khi thực hiện xong lệnh này mà không cần sửa nhé)
   + Đừng quên source devel/setup.bash
   + Và bắt đầu chạy mô phỏng robot hai bánh vi sai bằng câu lệnh: 
   > roslaunch diff_wheeled_robot_gazebo diff_wheeled_gazebo_full.launch
   + Lúc này màn hình gazebo sẽ hiện lên mô phỏng của robot 2 bánh vi sai, tất nhiên nếu tò mò có thể thử các file diff_wheeled_gazebo khác trong folder launch. Và sau đó thì thêm bao nhiêu obstacle hình vuông, tròn, trụ tùy ý để chuẩn bị chạy gmapping.
   + Mở một tab khác, và chạy câu lệnh:
   > roslaunch diff_wheeled_robot_gazebo gmapping.launch
   + Điều chỉnh tốc độ góc các thứ bằng bàn phím thì mở thêm terminal và chạy thêm câu lệnh: 
   > roslaunch diff_wheeled_robot_control keyboard_teleop.launch
   * Nếu mỗi lần chạy roslaunch trên một terminal mới mà lỗi thì nhớ luôn là mở terminal mới, make file lại bằng catkin_make và source devel/setup.bash để setup môi trường chạy cho các terminal*
   + Cuối cùng để xem được map đã được build chúng ta chạy rviz bằng câu lệnh ở một terminal mới rosrun rviz rviz
   + Chú ý set lại fix frame là odom thì sẽ thấy map, còn không thì base_link sẽ thấy cái obstacle mà laser_scan scan được nhé. Ở phía dưới chọn Add và thêm những yếu tố sau để hiện thị rviz là Map (nhớ chọn topic /map để xem được map đã build), RobotModel 
   + Và để lưu map thì dùng câu lệnh: rosrun map_server map_saver -f test (trong đó test là tên map mình vừa nghịch)
   + Một test nhỏ nếu muốn nghịch thêm sau khi chạy thành công bài toán chính thì có thẻ close rviz, và gmapping và diff_wheeled_robot_gazebo. Sau đó chạy llaij file launch của diff_wheeled_robot_gazebo_full.launch như hướng dẫn ở trên và mở một terminal mới không chạy gmapping mà chạy file amcl.launch Cái này là một trong những ứng dụng thành phần của navigation-stack package. Nó là autonomous navigation (Adaptive Monte Carlo Localization) tuy nhiên vẫn chưa ổn định như gmapping đâu và mở rviz để xem kết quả navigation.
  
## 3. Bài toán con: 
  + Cài urg_node for hokuyo_lidar: http://wiki.ros.org/urg_node (Dùng link source code trên git hoặc sudo apt install tuy nhiên ưu tiên clone source code)
  + Cài hector_slam để chạy slam building map dùng mỗi lidar chay: http://wiki.ros.org/hector_slam (recommend như trên)
  + Nói chung là recommend clone package source code trên ROS Wiki vào folder src của catkin_ws sau đó thực hiện đầy đủ catkin_make và source devel/setup.bash
  + Chạy file launch cho lidar: 
  > roslaunch urg_node urg_lidar.launch
  + Mở một terminal mới (cứ chạy roslaunch xong là mở thêm terminal thực hiện lại make và source nếu muốn launch tiếp) Còn kiểm tra lidar đã publish chưa (có thể không cần make và source): 
  > rostopic list
  + Check scan_message đã được publish (gửi đi):
  > rostopic echo /scan
  + Mở folder hector_mapping, tìm file mapping_default.launch sửa lại default ở hai dòng base_frame và odom_frame thành base_link vì hector_slam ko có base_footprint vì nó ko dùng robot mà là free odometry cho open system.
  + Sau đó kéo xuống cuối file yml, sửa lại dòng cuối cho node tf như sau:
  > <node pkg="tf" type="static_transform_publisher" name="base_to_broadcaster" args="0 0 0 0 0 0 base_link laser 100" />
  + Sau đó vào folder hector_slam_launch sửa file tutorial.launch cái phần use_sim_time = false vì mình dùng hệ thống thật chứ không phải mô phỏng.
  + Sau đó mở một terminal mới catkin_make và source devel/setup.bash và chạy hector_slam:
  > roslaunch hector_slam_launch tutorial.launch
  + Mở terminal mới lại mở rviz theo hướng dẫn ở trên, ở phần Add thêm LaserScan, sửa fix_frame thành /laser là hoàn thành có thể mở thêm Add Map nữa như bài toán chính tùy nhé.
