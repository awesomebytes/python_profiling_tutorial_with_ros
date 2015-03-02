cmake_minimum_required(VERSION 2.8.3)
project(rospy_tutorials)

find_package(catkin REQUIRED COMPONENTS std_msgs)


catkin_package(CATKIN_DEPENDS std_msgs)

# do not wildcard install files since the root folder of the package will contain a debian folder for releasing
catkin_install_python(PROGRAMS
  001_talker_listener/listener
  001_talker_listener/listener.py
  001_talker_listener/talker
  001_talker_listener/talker.py
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/001_talker_listener
)
install(FILES
  001_talker_listener/README
  001_talker_listener/talker_listener.launch
  001_talker_listener/listener.launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/001_talker_listener
)

