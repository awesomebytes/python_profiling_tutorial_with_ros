# python_profiling_tutorial_with_ros
Notes and test code to exemplify how to profile a python node (which is used in ROS)


The code to test is the python tutorials to keep it simple and some modified versions of it
to show interesting stuff.
All this has been tested in Ubuntu 12.04 32 bits.

# Install instructions

For using pprofile (https://github.com/vpelletier/pprofile):

    sudo pip install pprofile

For using line_profiler (https://github.com/rkern/line_profiler):

    sudo pip install line_profiler

Download `kcachegrind` to view results:

    sudo apt-get install kcachegrind



## Debug using pprofile
 
 You probably need a roscore running first:
    
    roscore
    
    
 Run python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener/talker.py
 
 And in another terminal run listener with pprofile:
 
    python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener$ pprofile listener.py --out listener.pprofile
    
 It will take a while to start, that's normal. Whenever you control+C the process (or it ends) you'll get the file with
 all the source code it used and some statistics on them. It will look like this (I searched for 
 "File: listener.py" in the output file) (You may want to use a lightweight text editor like sublime text to open the file
 as it gets very long easily):
 
    File: listener.py
    File duration: 0.00153661s (0.02%)
    Line #|      Hits|         Time| Time per hit|      %|Source code
    ------+----------+-------------+-------------+-------+-----------
         1|         0|            0|            0|  0.00%|#!/usr/bin/env python
    ~~~~~~~~~~~~~~~~~~~~~~~ Omitted here some license lines not interesting for us ~~~~~~~~~~~~~~~~~~~~~~~~~
        33|         0|            0|            0|  0.00%|#
        34|         0|            0|            0|  0.00%|# Revision $Id$
        35|         0|            0|            0|  0.00%|
        36|         0|            0|            0|  0.00%|## Simple talker demo that listens to std_msgs/Strings published 
        37|         0|            0|            0|  0.00%|## to the 'chatter' topic
        38|         0|            0|            0|  0.00%|
        39|         2|  0.000307798|  0.000153899|  0.00%|import rospy
    (call)|         1|      3.71281|      3.71281| 42.59%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/__init__.py:40 <module>
        40|         1|  2.31266e-05|  2.31266e-05|  0.00%|from std_msgs.msg import String
        41|         0|            0|            0|  0.00%|
        42|        21|  0.000114441|  5.44957e-06|  0.00%|def callback(data):
        43|        20|  0.000896215|  4.48108e-05|  0.01%|    rospy.loginfo(rospy.get_caller_id() + "I heard %s", data.data)
    (call)|        20|     0.150073|   0.00750364|  1.72%|# /usr/lib/python2.7/logging/__init__.py:1130 info
    (call)|        20|  0.000274658|  1.37329e-05|  0.00%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/names.py:312 get_name
        44|         0|            0|            0|  0.00%|
        45|         2|  1.78814e-05|   8.9407e-06|  0.00%|def listener():
        46|         0|            0|            0|  0.00%|
        47|         0|            0|            0|  0.00%|    # In ROS, nodes are uniquely named. If two nodes with the same
        48|         0|            0|            0|  0.00%|    # node are launched, the previous one is kicked off. The
        49|         0|            0|            0|  0.00%|    # anonymous=True flag means that rospy will choose a unique
        50|         0|            0|            0|  0.00%|    # name for our 'talker' node so that multiple talkers can
        51|         0|            0|            0|  0.00%|    # run simultaneously.
        52|         1|  5.07832e-05|  5.07832e-05|  0.00%|    rospy.init_node('listener', anonymous=True)
    (call)|         1|      1.96155|      1.96155| 22.50%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:189 init_node
        53|         0|            0|            0|  0.00%|
        54|         1|  4.02927e-05|  4.02927e-05|  0.00%|    rospy.Subscriber("chatter", String, callback)
    (call)|         1|    0.0357769|    0.0357769|  0.41%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/topics.py:452 __init__
        55|         0|            0|            0|  0.00%|
        56|         0|            0|            0|  0.00%|    # spin() simply keeps python from exiting until this node is stopped
        57|         1|  3.69549e-05|  3.69549e-05|  0.00%|    rospy.spin()
    (call)|         1|      3.00682|      3.00682| 34.49%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:117 spin
        58|         0|            0|            0|  0.00%|
        59|         1|  1.19209e-05|  1.19209e-05|  0.00%|if __name__ == '__main__':
        60|         1|  3.71933e-05|  3.71933e-05|  0.00%|    listener()
    (call)|         1|      5.00428|      5.00428| 57.40%|# listener.py:45 listener
    
 If we add a sleep, for example, in the callback function and we try again (giving it some seconds, 11s here, to execute and do something):
 
    pprofile listener_with_sleep.py --out listener_with_sleep.pprofile
    
 And we search again in listener_with_sleep.pprofile for "File: listener_with_sleep.py", we'll find:
 
    File: listener_with_sleep.py
    File duration: 0.00164151s (0.01%)
    Line #|      Hits|         Time| Time per hit|      %|Source code
    ------+----------+-------------+-------------+-------+-----------
         1|         0|            0|            0|  0.00%|#!/usr/bin/env python
    ~~~~~~~~~~~~~~~~~~~~~~~ Omitted here some license lines not interesting for us ~~~~~~~~~~~~~~~~~~~~~~~~~
        33|         0|            0|            0|  0.00%|#
        34|         0|            0|            0|  0.00%|# Revision $Id$
        35|         0|            0|            0|  0.00%|
        36|         0|            0|            0|  0.00%|## Simple talker demo that listens to std_msgs/Strings published 
        37|         0|            0|            0|  0.00%|## to the 'chatter' topic
        38|         0|            0|            0|  0.00%|
        39|         2|  0.000230789|  0.000115395|  0.00%|import rospy
    (call)|         1|      3.78909|      3.78909| 19.45%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/__init__.py:40 <module>
        40|         1|   2.5034e-05|   2.5034e-05|  0.00%|from std_msgs.msg import String
        41|         0|            0|            0|  0.00%|
        42|        12|  7.10487e-05|  5.92073e-06|  0.00%|def callback(data):
        43|        11|  0.000701904|  6.38095e-05|  0.00%|    rospy.loginfo(rospy.get_caller_id() + "I heard %s", data.data)
    (call)|        11|    0.0973785|   0.00885259|  0.50%|# /usr/lib/python2.7/logging/__init__.py:1130 info
    (call)|        11|  0.000152588|  1.38716e-05|  0.00%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/names.py:312 get_name
        44|        11|  0.000400782|  3.64347e-05|  0.00%|    rospy.sleep(1.0) # sleeping for one second for testing purposes
    (call)|        11|      13.2411|      1.20373| 67.96%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/timer.py:87 sleep
        45|         0|            0|            0|  0.00%|
        46|         2|  2.07424e-05|  1.03712e-05|  0.00%|def listener():
        47|         0|            0|            0|  0.00%|
        48|         0|            0|            0|  0.00%|    # In ROS, nodes are uniquely named. If two nodes with the same
        49|         0|            0|            0|  0.00%|    # node are launched, the previous one is kicked off. The
        50|         0|            0|            0|  0.00%|    # anonymous=True flag means that rospy will choose a unique
        51|         0|            0|            0|  0.00%|    # name for our 'talker' node so that multiple talkers can
        52|         0|            0|            0|  0.00%|    # run simultaneously.
        53|         1|  4.52995e-05|  4.52995e-05|  0.00%|    rospy.init_node('listener', anonymous=True)
    (call)|         1|      1.97952|      1.97952| 10.16%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:189 init_node
        54|         0|            0|            0|  0.00%|
        55|         1|  5.19753e-05|  5.19753e-05|  0.00%|    rospy.Subscriber("chatter", String, callback)
    (call)|         1|     0.069947|     0.069947|  0.36%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/topics.py:452 __init__
        56|         0|            0|            0|  0.00%|
        57|         0|            0|            0|  0.00%|    # spin() simply keeps python from exiting until this node is stopped
        58|         1|   4.1008e-05|   4.1008e-05|  0.00%|    rospy.spin()
    (call)|         1|      13.6458|      13.6458| 70.03%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:117 spin
        59|         0|            0|            0|  0.00%|
        60|         1|  1.21593e-05|  1.21593e-05|  0.00%|if __name__ == '__main__':
        61|         1|  4.07696e-05|  4.07696e-05|  0.00%|    listener()
    (call)|         1|      15.6954|      15.6954| 80.55%|# listener_with_sleep.py:46 listener

 In particular we can see that `Line #44`:
 
     Line #|      Hits|         Time| Time per hit|      %|Source code
     ------+----------+-------------+-------------+-------+-----------
         44|        11|  0.000400782|  3.64347e-05|  0.00%|    rospy.sleep(1.0) # sleeping for one second for testing purposes
     (call)|        11|      13.2411|      1.20373| 67.96%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/timer.py:87 sleep
    
 The call to `rospy.sleep(1.0)` was called 11 times and used 13.2411s of the running process, which is 67.96% of the time this file used.
 Every call took 1.20373s. You'll notice that sleeping for 1s but actually sleeping for 1.2s looks quite bad, but this just happens because of the
 overhead the profiling gives for every call/line. You'll notice that imports also take a lot longer because of this. If I run it for longer you'll see
 that the percentages of time make more sense.
 
 I ran it for 130s:
 
    pprofile listener_with_sleep.py --out listener_with_sleep.pprofile2
    
 And the new output looks like:
 
    File: listener_with_sleep.py
    File duration: 0.00880742s (0.01%)
    Line #|      Hits|         Time| Time per hit|      %|Source code
    ------+----------+-------------+-------------+-------+-----------
         1|         0|            0|            0|  0.00%|#!/usr/bin/env python
    ~~~~~~~~~~~~~~~~~~~~~~~ Omitted here some license lines not interesting for us ~~~~~~~~~~~~~~~~~~~~~~~~~
        33|         0|            0|            0|  0.00%|#
        34|         0|            0|            0|  0.00%|# Revision $Id$
        35|         0|            0|            0|  0.00%|
        36|         0|            0|            0|  0.00%|## Simple talker demo that listens to std_msgs/Strings published 
        37|         0|            0|            0|  0.00%|## to the 'chatter' topic
        38|         0|            0|            0|  0.00%|
        39|         2|  0.000241041|  0.000120521|  0.00%|import rospy
    (call)|         1|      3.62363|      3.62363|  2.79%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/__init__.py:40 <module>
        40|         1|   1.5974e-05|   1.5974e-05|  0.00%|from std_msgs.msg import String
        41|         0|            0|            0|  0.00%|
        42|       103|  0.000472307|  4.58551e-06|  0.00%|def callback(data):
        43|       102|    0.0045867|  4.49676e-05|  0.00%|    rospy.loginfo(rospy.get_caller_id() + "I heard %s", data.data)
    (call)|       102|      0.78971|   0.00774225|  0.61%|# /usr/lib/python2.7/logging/__init__.py:1130 info
    (call)|       102|   0.00141764|  1.38984e-05|  0.00%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/names.py:312 get_name
        44|       102|    0.0033133|  3.24834e-05|  0.00%|    rospy.sleep(1.0) # sleeping for one second for testing purposes
    (call)|       102|      122.895|      1.20485| 94.63%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/timer.py:87 sleep
        45|         0|            0|            0|  0.00%|
        46|         2|   1.3113e-05|  6.55651e-06|  0.00%|def listener():
        47|         0|            0|            0|  0.00%|
        48|         0|            0|            0|  0.00%|    # In ROS, nodes are uniquely named. If two nodes with the same
        49|         0|            0|            0|  0.00%|    # node are launched, the previous one is kicked off. The
        50|         0|            0|            0|  0.00%|    # anonymous=True flag means that rospy will choose a unique
        51|         0|            0|            0|  0.00%|    # name for our 'talker' node so that multiple talkers can
        52|         0|            0|            0|  0.00%|    # run simultaneously.
        53|         1|  2.88486e-05|  2.88486e-05|  0.00%|    rospy.init_node('listener', anonymous=True)
    (call)|         1|      1.89639|      1.89639|  1.46%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:189 init_node
        54|         0|            0|            0|  0.00%|
        55|         1|  4.22001e-05|  4.22001e-05|  0.00%|    rospy.Subscriber("chatter", String, callback)
    (call)|         1|    0.0514629|    0.0514629|  0.04%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/topics.py:452 __init__
        56|         0|            0|            0|  0.00%|
        57|         0|            0|            0|  0.00%|    # spin() simply keeps python from exiting until this node is stopped
        58|         1|  4.69685e-05|  4.69685e-05|  0.00%|    rospy.spin()
    (call)|         1|      124.303|      124.303| 95.71%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:117 spin
        59|         0|            0|            0|  0.00%|
        60|         1|  5.96046e-06|  5.96046e-06|  0.00%|if __name__ == '__main__':
        61|         1|   4.1008e-05|   4.1008e-05|  0.00%|    listener()
    (call)|         1|      126.251|      126.251| 97.21%|# listener_with_sleep.py:46 listener

You'll see that `import rospy` takes only 2.79% of the time and the rest (97.21%) was taken by the actual execution of the node `listener()`.
Now from that percentage 94.63% was used in the `rospy.sleep(1.0)` call and a significant bit (1.46%) in the `rospy.init_node('listener', anonymous=True)`.

 Let's make a node that computes something and see if it's easily detected.
 
    pprofile listener_with_computation.py --out listener_with_computation.pprofile
     
 Open the output and search for "File: listener_with_computation.py":
 
    File: listener_with_computation.py
    File duration: 3.35888s (5.41%)
    Line #|      Hits|         Time| Time per hit|      %|Source code
    ------+----------+-------------+-------------+-------+-----------
         1|         0|            0|            0|  0.00%|#!/usr/bin/env python
        33|         0|            0|            0|  0.00%|#
        34|         0|            0|            0|  0.00%|# Revision $Id$
        35|         0|            0|            0|  0.00%|
        36|         0|            0|            0|  0.00%|## Simple talker demo that listens to std_msgs/Strings published 
        37|         0|            0|            0|  0.00%|## to the 'chatter' topic
        38|         0|            0|            0|  0.00%|
        39|         2|  0.000226974|  0.000113487|  0.00%|import rospy
    (call)|         1|      3.79679|      3.79679|  6.12%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/__init__.py:40 <module>
        40|         1|  2.40803e-05|  2.40803e-05|  0.00%|from std_msgs.msg import String
        41|         1|  1.38283e-05|  1.38283e-05|  0.00%|import math
        42|         0|            0|            0|  0.00%|
        43|       469|   0.00324583|  6.92075e-06|  0.01%|def expensive_operation(input_list):
        44|       468|   0.00402856|  8.60803e-06|  0.01%|    result = 0.0
        45|     98678|     0.805604|  8.16397e-06|  1.30%|    for elem in input_list:
        46|     98210|     0.847585|  8.63033e-06|  1.37%|        result += math.sqrt(elem)
        47|       468|   0.00363588|  7.76898e-06|  0.01%|    return result
        48|         0|            0|            0|  0.00%|
        49|       469|    0.0030582|  6.52067e-06|  0.00%|def callback(data):
        50|         0|            0|            0|  0.00%|    # Commenting out prints as they are way more expensive than the computations done
        51|         0|            0|            0|  0.00%|    # rospy.loginfo(rospy.get_caller_id() + "I heard %s", data.data)
        52|         0|            0|            0|  0.00%|    # Do something that actually computes something
        53|       468|   0.00446057|  9.53114e-06|  0.01%|    very_important_accumulator = []
        54|     98678|     0.846788|  8.58132e-06|  1.37%|    for i in range(len(data.data) * 10):
        55|     98210|     0.823188|  8.38191e-06|  1.33%|        very_important_accumulator.append(i)
        56|       468|    0.0168076|  3.59136e-05|  0.03%|    very_important_result = expensive_operation(very_important_accumulator)
    (call)|       468|      1.66408|   0.00355574|  2.68%|# listener_with_computation.py:43 expensive_operation
        57|         0|            0|            0|  0.00%|    # rospy.loginfo("Very important result is: " + str(very_important_result))
        58|         0|            0|            0|  0.00%|
        59|         2|  1.88351e-05|  9.41753e-06|  0.00%|def listener():
        60|         0|            0|            0|  0.00%|
        61|         0|            0|            0|  0.00%|    # In ROS, nodes are uniquely named. If two nodes with the same
        62|         0|            0|            0|  0.00%|    # node are launched, the previous one is kicked off. The
        63|         0|            0|            0|  0.00%|    # anonymous=True flag means that rospy will choose a unique
        64|         0|            0|            0|  0.00%|    # name for our 'talker' node so that multiple talkers can
        65|         0|            0|            0|  0.00%|    # run simultaneously.
        66|         1|  4.72069e-05|  4.72069e-05|  0.00%|    rospy.init_node('listener', anonymous=True)
    (call)|         1|      1.97497|      1.97497|  3.18%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:189 init_node
        67|         0|            0|            0|  0.00%|
        68|         1|  4.88758e-05|  4.88758e-05|  0.00%|    rospy.Subscriber("chatter", String, callback)
    (call)|         1|     0.045511|     0.045511|  0.07%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/topics.py:452 __init__
        69|         0|            0|            0|  0.00%|
        70|         0|            0|            0|  0.00%|    # spin() simply keeps python from exiting until this node is stopped
        71|         1|  4.50611e-05|  4.50611e-05|  0.00%|    rospy.spin()
    (call)|         1|      56.2169|      56.2169| 90.62%|# /opt/ros/hydro/lib/python2.7/dist-packages/rospy/client.py:117 spin
        72|         0|            0|            0|  0.00%|
        73|         1|  1.19209e-05|  1.19209e-05|  0.00%|if __name__ == '__main__':
        74|         1|  4.31538e-05|  4.31538e-05|  0.00%|    listener()
    (call)|         1|      58.2375|      58.2375| 93.88%|# listener_with_computation.py:59 listener
 
  We see that the `expensive_operation()` call takes 2.68% of the time.
  
  Now for more real life examples, for python files that are launched by roslaunch .launch files for example, we would need to launch
  the node in the same conditions than it's ran in the real environment. As an example you can launch listener with the provided
  listener.launch.
  
    roslaunch listener.launch
  
  Then you can do:
  
    ps aux | grep listener
    
  And you'll see something like:
  
    1000027   6708 32.0  0.1  81440 10404 ?        Ssl  17:44   0:00 python /home/YOURUSER/YOURWORKSPACEPATH/python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener/listener chatter:=chatter __name:=listener __log:=/home/YOURHOME/.ros/logs/7c6e55f0-c0cd-11e4-85d8-e0cb4e1f7c63/listener-1.log

  The important part is:
  
    /home/YOURUSER/YOURWORKSPACEPATH/python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener/listener chatter:=chatter __name:=listener
    
 Which gives you the parameters that the node needs, the remaps of topics, name of the running node (may be important in some systems)... and if you need
 parameters to be uploaded to the param server you should upload them by hand most probably. So you only need to do:
 
    pprofile  --out /YOURPATH/youroutputfile.pprofile /home/YOURUSER/YOURWORKSPACEPATH/python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener/listener chatter:=chatter __name:=listener
    
 By the way, the extension .pprofile, I just made it up.
 
 Also you can use the `callgrind` format as output and open it with `kcachegrind`:
 
    pprofile  --format callgrind --out /home/YOURUSER/YOURWORKSPACEPATH/python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener/listener.callgrindoutput /home/YOURUSER/YOURWORKSPACEPATH/python_profiling_tutorial_with_ros/rospy_tutorials/001_talker_listener/listener chatter:=chatter __name:=listener

 Open it:
 
    kcachegrind listener.callgrindoutput
    
 [kcachegrind screenshot](https://raw.githubusercontent.com/awesomebytes/python_profiling_tutorial_with_ros/master/kcacegrind_screenshot.png)
 
 The interpretation of the data with `kcachegrind` may be more complicated, here you have the official documentation: https://lbtwiki.cern.ch/bin/view/Online/Kcachegrind. 
 I found out that ordering by the Self collum was the most useful for me. What it means?
 
    Incl. is the inclusive cost of a function, this means the function itself and all it's including functions. Self is only the cost of the function itself.
    
 Why do I say it's more complicated to interpret that the first method? Because it includes all the boilerplate of
 threading, serializing, timing, networking of the ROS node, which is A LOT and it usually involves most of the execution
 time of a node.

## Debug using line_profiler 


# Other options revised

Note: this was painful to get it to work, that's why it's presented aside.

Graphical python profiler

http://www.pyvmmonitor.com/download.html

To use it:

    sudo pip install yappi

Then 

    ./pyvmmonitor-ui

I got:

    Error:
    stderr: gdb: unrecognized option '--nh'


Which is because:

https://github.com/google/pyringe/issues/13
--nh flag was introduced in gdb 7.6.1 stable (actually 7.6-1 unstable), but Ubuntu precise's official repo have gdb 7.4

Download last gdb:
http://ftp.gnu.org/gnu/gdb/

    wget http://ftp.gnu.org/gnu/gdb/gdb-7.9.tar.gz

Extract... and compile:

    ~/Downloads/gdb-7.9$ ./configure --prefix=/home/YOURUSER/gdb-7.9
    make
    make install exec_prefix=~/gdb-7.9/

(this is very ugly, I know)

    sudo mv /usr/bin/gdb /usr/bin/gdb.backup
    sudo ln -s ~/gdb-7.9/bin/gdb /usr/bin/gdb

You can open it with (once again):

    ./home/YOURUSER/Downloads/pyvmmonitor-ui

Then to any code you want to profile while on runtime you must add at the start:

    import sys
    sys.path.append('/home/YOURUSER/Downloads/pyvmmonitor/public_api')
    import pyvmmonitor
    pyvmmonitor.connect()

And your running pyvmmonitor will attach to the process automaticly. Looks like this:
<screenshot>

You can save a session of running profiling data hitting the play button near `yappi`. Then you can
check it's data in the same interface when you press pause. Or you can save that data and open it with
`kcachegrind`.

