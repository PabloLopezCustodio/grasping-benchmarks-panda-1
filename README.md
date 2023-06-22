# grasping-benchmarks-panda


## Modifications of this fork
This fork allows the use of the latest version of Dexnet. It also allows Graspnet to work without a graphics interface, and fixes other problems faced when running the containers and libraries at CMS

* docker/run.sh: Doesn't roslaunch the service automatically, which was giving me problems
* docker/build_images/Makefile: A different image is chosen in order to install ROS Noetic wuth Python3
* docker/dexnet/Dockerfile: installs ROS Noetic, fixes ownership
* grasping_benchmarks/dexnet/dexnet_grasp_planner.py: fixes functions from SciPy which were updated in the library's latest version
* docker/dexnet/Dockerfile: pulls forked graspnet library which works without graphics interface
* grasping_benchmarks/graspnet/graspnet_grasp_planner.py: allows use without graphics interface, discards use of aruco marker
