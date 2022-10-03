# Yocto Build
Xây dựng Yocto Distro
=====================
Yocto Project là dự án mã nguồn mở tập trung vào phát triển hệ điều hành Linux dành cho hệ thống nhúng.
Hiện tại Yocto chỉ hỗ trợ các phiên bản Ubuntu 16.04, 18.04 và 20.04.

Cài các gói cần thiết để build image cho headless system (không có màn hình, GUI, ngoại vi,...):
----------------------------------------------------------------------------------------
	~$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev

Yêu cầu về phiên bản Git, tar, Python, gcc:
-------------------------------------------
- Git 1.8.3.1 trở lên
- tar 1.28 trở lên
- Python 3.5.0 trở lên
- gcc 5.0 trở lên

Tạo folder chứa Yocto:
-----------------------
	~$ mkdir yocto
	~$ cd yocto

Clone Poky về (hiện tại phiên bản mới nhất là Kirkstone, có thể theo dõi thông tin của các phiên bản tại https://wiki.yoctoproject.org/wiki/Releases):
--------------------------------------------------------------------------------------------
	~$ git clone git://git.yoctoproject.org/poky -b kirkstone

Vì Poky mặc định không hỗ trợ cho Raspberry Pi (chỉ hỗ trợ Texas Instruments Beaglebone và Freescale MPC8315E-RDB) nên ta phải tải thêm các lớp hỗ trợ board mạch (BPS) của Raspberry Pi:

	~$ cd poky 
	~$ git clone -b kirkstone git://git.yoctoproject.org/meta-raspberrypi 

Bắt đầu build:
---------------
- Đầu tiên, tại Poky chạy:

	~$ source oe-init-build-env

Sau khi chạy xong đường dẫn sẽ tự chuyển về thư mục build của Yocto.
Trước khi tiến hành build, vào thư mục build/conf  và chỉnh sửa 2 file bblayers.conf và local.conf.
Trong file bbblayers.conf thêm đường dẫn của BPS cho Raspberry Pi vừa clone bên trên vào. Sau khi chỉnh sửa file sẽ có nội dung kiểu như sau:

	# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
	# changes incompatibly
	POKY_BBLAYERS_CONF_VERSION = "2"

	BBPATH = "${TOPDIR}"
	BBFILES ?= ""

	BBLAYERS ?= " \
	  /home/tin/kirkstone/poky/meta \
	  /home/tin/kirkstone/poky/meta-poky \
	  /home/tin/kirkstone/poky/meta-yocto-bsp \
	  /home/tin/kirkstone/poky/meta-raspberrypi \ # mới thêm vào
	  "

Còn trong file local.conf, vì mặc định Yocto không hỗ trợ board Raspberry Pi nên ta sẽ phải thêm nó vào. Thêm vào 1 dòng:

	MACHINE ?= "raspberrypi3"

Ở đây đang build trên Raspberry Pi 3, nếu build cho Raspberry Pi 4 sẽ là:

	MACHINE ?= "raspberrypi4" 

Thay đổi tương tự cho các phiên bản khác.
Để thuận tiện cho quá trình debug sau build và tránh phải sửa file sau mỗi lần build, ta kích hoạt UART cho board từ bước này, thêm vào 2 dòng:

	ENABLE_UART = "1" 
	ENABLE_KGBD = "1"

Sau khi xong, file local.conf sẽ có nội dung kiểu như sau:

	#MACHINE ?= "qemuarm"
	#MACHINE ?= "qemuarm64"
	#MACHINE ?= "qemumips"
	#MACHINE ?= "qemumips64"
	#MACHINE ?= "qemuppc"
	#MACHINE ?= "qemux86"
	#MACHINE ?= "qemux86-64"
	#
	# There are also the following hardware board target machines included for
	# demonstration purposes:
	#
	#MACHINE ?= "beaglebone-yocto"
	#MACHINE ?= "genericx86"
	#MACHINE ?= "genericx86-64"
	#MACHINE ?= "edgerouter"
	#
	# This sets the default machine to be qemux86-64 if no other machine is selected:
	MACHINE ?= "raspberrypi3" # mới thêm vào
	ENABLE_UART = "1"   # mới thêm vào
	ENABLE_KGBD = "1"  # mới thêm vào

Bây giờ tiến hành build, sau khi chạy xong lệnh source lúc nãy, ta có thể thấy một đoạn hướng dẫn dùng bitbake:

	bitbake <target>
	
Ta cũng làm tương tự như vậy:

	~$ bitbake rpi-test-image 

Một số Image thông dụng là: core-image-base, core-image-minimal, rpi-basic-image, rpi-test-image,...
Vì phần BPS của nhánh Kirkstone không có recipe của rpi-basic-image nên ta sẽ build tạm rpi-test-image, nếu muốn có thể tải thêm recipe của rpi-basic-image từ nhánh khác như jethro bỏ vào folder /poky/meta-raspberrypi/recipes-core/images và build.

Trong quá trình build thường sẽ xảy ra lỗi, ta sẽ đợi build xong sau đó mới tiến hành sửa lỗi và build tiếp.
Giả sử sau khi build xong gặp lỗi như sau:

	ERROR: Task (/home/.../imx-yocto-bsp/sources/poky/meta/recipes-graphics/vulkan/vulkan_1.0.65.2.bb:do_compile) failed with exit code '1'
Lúc này cần xác định tên file bị lỗi, như ở lỗi trên là do file vulkan.
Một ví dụ khác:

	ERROR: Task (/home/.../imx-yocto-bsp/sources/meta-fsl-bsp-release/imx/meta-bsp/recipes-graphics/opencv/opencv_4.0.1.imx.bb:do_compile) failed with exit code '1'

Thì lúc này sẽ là lỗi file opencv.
Sau khi đã biết file nào bị lỗi ta xóa đi file đó đi và build riêng bằng cách:

	~$ bitbake -c cleanall <filegặplỗi>

Ví dụ:
	~$ bitbake -c cleanall vulkan (ví dụ 1)
Hoặc
	~$ bitbake -c cleanall opencv (ví dụ 2)

Và bắt đầu build riêng file lỗi:

	~$ bitbake <filegặplỗi>

Ví dụ:
	~$ bitbake vulkan (ví dụ 1)
Hoặc:
	~$ bitbake opencv (ví dụ 2)

Sau khi xong thì tiến hành build tiếp tục file image ban nãy:

	~$ bitbake rpi-test-image 

Quá trình này lặp lại cho đến khi build thành công, không còn xuất hiện lỗi. File sau khi build thành công sẽ được lưu trong thư mục poky/build/tmp/deploy/images/<tênboard>
Sau khi build thành công ta tiến hành flash file image này vào micro sd card bằng RPI Imager. Trong RPI Imager chọn Choose OS -> Use Custom. Vì file output không phải là file đuôi image nên sẽ không xuất hiện trong lựa chọn của RPI Imager lúc này chuyển từ lọc image files sang tất cả các file, chọn tới thư mục poky/build/tmp/deploy/images/<tênboard>. File output sẽ có định dạng .wic.bz2 ví dụ rpi-test-image-raspberrypi3.wic.bz2. Chọn file output, chọn micro sd card cần nạp và ghi vào.

Thông thường sau bước này ta sẽ phải chỉnh sửa file config.txt và cmdline.txt trong ổ Boot (micro sd card). Nhưng vì đã chỉnh sửa file local.conf trước đó nên ta chỉ cần kiểm tra file cmdline.txt xem đã có dòng khởi tạo baud rate là 115200 hay chưa, nếu chưa sẽ thêm vào:

	console=serial0,115200

Sau đó gắn lại micro sd card vào Raspberry Pi, cắm mạch nạp vào các chân: 

- Chân GND vào Pin 6
- Chân Rx của mạch nạp vào Pin 8
- Chân Tx của mạch nạp vào Pin 10

Cài Screen cho Ubuntu:

	~$ sudo apt-get screen

Trước khi cắm mạch nạp vào máy tính, dùng câu lệnh:

	~$ sudo ls /dev/ttyUSB* (để kiểm tra các thiết bị ttyUSB hiện tại.)

Sau đó cắm mạch nạp vào và sử dụng lại câu lệnh ~$ sudo ls /dev/ttyUSB* kiểm tra thay đổi. ttyUSB vừa xuất hiện là của mạch nạp (thông thường là ttyUSB0)

Chạy màn hình debug:

	~$ sudo screen /dev/ttyUSB0 115200

Cắm nguồn cho Raspberry Pi, đợi vài giây, nếu thành công sẽ thấy Raspberry Pi bắt đầu Boot.
Tên người dùng mặc định là root.





