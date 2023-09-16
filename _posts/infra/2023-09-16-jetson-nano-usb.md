---
layout: post 
title: Jetson nano install 및 boot usb(with black sreen)
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Jetson, Infra]
categories : [Infra]
toc: true
---


### TRY  
원래 vscode 서버용 및 간단한 api 서버, 장난감용으로 쓰던 Jetson 시리즈를 팔았다가 다시 사용하고 싶어져서 구매했다.  
처음에 jetson image를 sd card에 설치를 하고 usb로 부팅하려 했는데 문제가 생겼다.  
그후 다시 sd card를 flash 하자..  

### CATCH  

먼저 처음 flash 하던 방법은 다음과 같다.  

* 설치 
    1. [Etcher](https://etcher.balena.io/)와 [Jetson Nano Developer Kit SD Card Image-4.6.1](https://developer.nvidia.com/embedded/downloads#?search=jetson%20nano)을 다운받는다.
    2. etcher를 이용해 sdcard를 flash 한다.
    3. 사용한다.  

그후 [rootOnUsb](https://github.com/JetsonHacksNano/rootOnUSB)를 이용해 `너무나도 느린 sd 카드`가 아닌 `ssd`를 root으로 사용하려 했다.  
근데 NVIDA 로고 뒤로 넘어가지않았다.  

찾아보니 jetpack 4.5부터 usb를 통해 부팅할 수 있게 바뀌었나보다. ([bootFromUSB](https://github.com/jetsonhacks/bootFromUSB))  
물론 안넘어간게 이것 떄문은 아닐 것 이다.  
예전에 사용할땐 sd카드를 꽂아놓은 상태에서 sdcard 를 통해 usb로 부팅되게 설정을 바꿔줘야 했기에 usb를 제거하거나 sd카드를 제거하면 부팅이 안됐었는데..  
아무튼 bootFromUSB를 사용하기위해 다시 sd카드를 Flash를 하고 확인하려는데..

* black sreean  
    정말 까만 화면엔 아무 글자도, 그림도 나오지 않았다.  
    그냥 까만화면만 보여주는 모니터엔 내 얼굴만 보였다.  
    [Nvidia Jetson forum](https://forums.developer.nvidia.com/c/agx-autonomous-machines/jetson-embedded-systems/70)을 정말 열심히 뒤져봤는데 다른 사람들은 터미널에서 에러 코드가 보이거나 nvidia 로고라도 보였는데 난 정말 아무것도 안나왔다.  
    hdmi를 확인해봤지만 문제 없었다.  
    나처럼 완전 아무것도 안나오는 사람들도 있었지만 다들 그냥 다시 flash 해보라는 소리밖에 없었다.  
    etcher나 다른 걸 이용해 열심히 flash 해봤지만 똑같았다.  

    내가 해결방법은 다음과 같았다.  
    
    1. 윈도우에 VMWare를 설치하여 우분투18.04를 올린다.(집에 리눅스 pc가 없기 떄문)
    2. vm에서 sdk manager를 다운받는다
    3. jetson을 usb에 연결해 vm과 연결한다.(jumper를 force recovery 와 gnd에 장착한 후 usb를 연결하였다.)
    4. sdk manager를 통해 flash한다.(menual install)
    5. sd card를 제거해 jetson에 연결한다.
    6. nvidia 로고를 즐긴다.

    이와 같이 윈도우에 우분투를 올려서 해결했다.  
    다만 Virtual box를 사용하면 flash가 98~99%에서 멈춰버린다.  
    포럼에도 이같은 문제가 제기되어 있다, vmware를 사용하자.  

이제 bootFromUSB를 이용해 SSD를 root으로 사용하려고 했다.  

* bootFromUSB  
    1. $ git clone https://github.com/jetsonhacks/bootFromUSB.git
    2. ssd 포맷 (ext4, partitioning은 GUID Partition Table) 
    3. $ cd bootFromUSB
    4. $ ./copyRootToUSB.sh 
    5. $ sudo gedit extlinux.conf
    6. 다음 LABEL primary를 복사해 밑에 붙여넣고 sdcard로 수정했다.  
        ```shell
            TIMEOUT 30
            DEFAULT primary

            MENU TITLE L4T boot options

            LABEL primary
                MENU LABEL primary kernel
                LINUX /boot/Image
                INITRD /boot/initrd
                APPEND ${cbootargs} quiet root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4 console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0 

            LABEL sdcard
                MENU LABEL primary kernel
                LINUX /boot/Image
                INITRD /boot/initrd
                APPEND ${cbootargs} quiet root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4 console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0 

            # When testing a custom kernel, it is recommended that you create a backup of
            # the original kernel and add a new entry to this file so that the device can
            # fallback to the original kernel. To do this:
            #
            # 1, Make a backup of the original kernel
            #      sudo cp /boot/Image /boot/Image.backup
            #
            # 2, Copy your custom kernel into /boot/Image
            #
            # 3, Uncomment below menu setting lines for the original kernel
            #
            # 4, Reboot

            # LABEL backup
            #    MENU LABEL backup kernel
            #    LINUX /boot/Image.backup
            #    INITRD /boot/initrd
            #    APPEND ${cbootargs}
        ```
    7. 아까 boodFromUSB의 `./partUUID.sh` 를 실행해 PARTUUID를 알아낸다.
        ```
        PARTUUID of Partition: /dev/sda1
        5fb9d375-541c-4139-a71d-836a815d421a

        Sample snippet for /boot/extlinux/extlinux.conf entry:
        APPEND ${cbootargs} root=PARTUUID=5fb9d375-541c-4139-a71d-836a815d421a rootwait rootfstype=ext4
        ```  
        맨 아래줄의 root 부분을 사용할 것이다.  
    8. primary의 root을 7.에서 얻는 값으로 변경한다. (root 부분)
        ```shell
            TIMEOUT 30
            DEFAULT primary

            MENU TITLE L4T boot options

            LABEL primary
                MENU LABEL primary kernel
                LINUX /boot/Image
                INITRD /boot/initrd
                                            # 이부분
                APPEND ${cbootargs} quiet root=PARTUUID=5fb9d375-541c-4139-a71d-836a815d421a rw rootwait rootfstype=ext4 console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0 

            LABEL sdcard
                MENU LABEL primary kernel
                LINUX /boot/Image
                INITRD /boot/initrd
                APPEND ${cbootargs} quiet root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4 console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0 

            # When testing a custom kernel, it is recommended that you create a backup of
            # the original kernel and add a new entry to this file so that the device can
            # fallback to the original kernel. To do this:
            #
            # 1, Make a backup of the original kernel
            #      sudo cp /boot/Image /boot/Image.backup
            #
            # 2, Copy your custom kernel into /boot/Image
            #
            # 3, Uncomment below menu setting lines for the original kernel
            #
            # 4, Reboot

            # LABEL backup
            #    MENU LABEL backup kernel
            #    LINUX /boot/Image.backup
            #    INITRD /boot/initrd
            #    APPEND ${cbootargs}
        ```  
    9. 저장, pc 종료 후 sdcard 제거 및 재시작 해준다.  

    이후 NVIDIA 로고에서 넘어가지않았는데 usb에 꽂아두었던 키보드와 마우스를 제거한 후 재부팅하니 넘어갔다.  


### FINALLY  

아 오랜만에 이것저것 만져보려니 재미도 있었지만 왜 안되는지 잘 모르겠어서 힘들었다.  
중간에 그냥 다시 팔까 고민도 했다.  
nvidia 포럼에도 다들 비슷한 얘기만 있어서 크게 도움되진 않았던 것 같다.  
그래도 잘 설치해서 다행이다.  

끝

---
