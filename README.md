OpenWrt-FL2440 �����ʼ�
===
---------------------------------
ͬ����openwrt�ٷ�Ƕ��ʽ�����汾��֧��������Ӷ� fl2440 ������� porting ������

���˵����μ��ٷ��ĵ����òִ������ο�ѧϰʹ��.

�ٷ�Դ���£�

SVN�֣�[svn://svn.openwrt.org.cn/dreambox/branches/openosom](https://dev.openwrt.org.cn/browser/branches/openosom)

GIT�֣�[git://github.com/openosom/backfire_10.03.1](https://github.com/openosom/backfire_10.03.1)

��������Ҫ���޸Ĺ��̼�����

1. ��װ���뻷��
---------------------------------
    
    $ sudo apt-get install subversion build-essential libncurses5-dev zlib1g-dev gawk git ccache gettext libssl-dev xsltproc file

2. �򵥵ı�����̷���
---------------------------------

    1) ��װ device, packages, managements, xwrt, luci ������齨

        a) ͬ�������齨���� feeds update �Ĺ���

                $ ./scripts/feeds update -a

            ����������У�feeds ͨ�� feeds.conf.default �ļ������õĸ�������Ļ�ȡ��ʽ����ȡԴ��ͬ����Ӧ�����ݣ�ͬ����ʽ��Ҫ�� src-svn, src-cpy, src-link, src-git, src-gitsvn, src-bzr, src-hg �� 7 �з�ʽ(��ϸ�μ� scritps/feeds �ļ�)���������ص� feeds Ŀ¼�½��д�ţ�ͬʱ�����һЩ��ʱ�ļ������� devices ���� devices.tmp, devices.index��

        b) ��װ�����齨���� feeds install �Ĺ���

                $ ./scripts/feeds install -a

            �ù���ʵ�����ǽ� feeds update ͬ���µ��齨 copy �� package/feeds Ŀ¼�½��д�š��ڰ�װ��Щ�ļ�֮ǰ�������˱���ʱ����Ҫ��һЩ��Ҫ���ļ��У��������£�

                mkdir -p ./staging_dir/toolchain-arm_v4t_gcc-4.3.3+cs_eglibc-2.8_eabi
                cd ./staging_dir/toolchain-arm_v4t_gcc-4.3.3+cs_eglibc-2.8_eabi
                mkdir -p stamp lib usr/include usr/lib

            ����Ϊ֮����뽻����빤�ߺ��ļ�ϵͳ��׼����
            
    2) �޸ı���˵�
        
        make menuconfig

    3) ִ�б���

        make V=99   # ��ӡ���б������
        ����
        make V=s  # ����ӡ�κα��������Ϣ( s �� silent ����˼)

        �� V ��������Ķ��壬��ʵ���� openwrt ����ϵͳ�е�һ����Ҫ��������ʹ������ include/verbose.mk �ļ��н��д���ģ����£�

            ifeq ("$(origin V)", "command line")
                KBUILD_VERBOSE:=$(V)
            endif
        
        ͨ��������ʽ�� V �������ݸ� KBUILD_VERBOSE ������Ȼ���������н����жϣ������ 99 �ͻᶨ���������ݣ�
  
            SUBMAKE=$(MAKE) -w
            define MESSAGE
                printf "%s\n" "$(1)"
            endef
            

2. ��� Boot Loader 
---------------------------------
    
    �� openwrt ����Ҫ��ͨ�� feeds/device ���������豸����ص����ݣ����а����� qemu �������ص����ݣ�(���ʼ�ǽ� uboot ��ض���Ҳ�Ƿ��� menuconfig �˵������ Devices �Ӳ˵��У�������˲鿴�ѽ���Ӧ�����ƶ��� Boot Loaders �˵���) 

    Ϊ�˷��������������Լ�����һ�� git server ������Ϊ device feeds ��ȡ��Դ��Ȼ��ͨ���޸� feed.conf �ļ����� ./scripts

    1) �� feeds/device Ŀ¼�´��� uboot-fl2440 Ŀ¼, �������Ӧ�� Makefile �ļ���
    2) �޸���Ӧ�� Makefile �ļ��Ӷ�ʵ���Զ���� boot loader ��װ�ع��̣��Լ��������ã��������£�
        
        a) Դ������

            # ������  
            PKG_NAME:=uboot-fl2440
            # �汾��Ϣ
            PKG_VERSION:=2010.09
            PKG_RELEASE:=1

            # ����ѹ���ļ�
            PKG_SOURCE:=u-boot-2010.03-fl2440.tar.bz2
            # ��ѹ���ļ�
            PKG_SOURCE_SUBDIR:=u-boot-2010.09-fl2440
            # ����Ŀ¼
            PKG_BUILD_DIR=$(KERNEL_BUILD_DIR)/$(PKG_SOURCE_SUBDIR)
            # ����ͬ��·��
            PKG_SOURCE_URL:=git://github.com/iAlios/fl2440-uboot-2010.09.git
            # ����ͬ����ʽ
            PKG_SOURCE_PROTO:=git
            # ����汾(ָ�����ύ�Ľڵ� commit ID)
            PKG_SOURCE_VERSION:=7877290e987a95dc834e686a059ab5ceb04fb714

        b) menuconfig ����
                    
            define Package/$(PKG_NAME)
              # ���ò˵�����
              TITLE:=for fl2440(iAlios)
              # ���ø��˵�������
              CATEGORY:=Boot Loaders
              # ���÷�������
              SECTION:=uboot
              # ���ñ�������
              DEPENDS:=@TARGET_s3c24xx 
              URL:=http://www.denx.de/wiki/U-Boot
            endef

        c) ��������

            define Build/Prepare
                # ��ѹ�ļ�
                $(PKG_UNPACK)
                # ��װ patch
                $(Build/Patch)
                # ɾ������Ҫ���м��ļ�
                $(FIND) $(PKG_BUILD_DIR) -name .svn | $(XARGS) rm -rf
            endef

            define Build/Compile
                # ��������
                $(MAKE) -C $(PKG_BUILD_DIR) fl2440_config
                # ����
                $(MAKE) -C $(PKG_BUILD_DIR) CROSS_COMPILE=$(TARGET_CROSS)
            endef

        d) ��װ����

            define Package/$(PKG_NAME)/install
                $(INSTALL_DIR) $(BIN_DIR)
                dd if=$(PKG_BUILD_DIR)/u-boot.bin of=$(BIN_DIR)/$(PKG_NAME).bin bs=128k conv=sync
            endef

3. ����� CPU �ܹ�
---------------------------------

   ������ kernel ��ص����ݶ��Ǵ���� target/linux Ŀ¼�¡�

    1) �� openwrt ����ϵͳ����ͨ�� Config.in �ļ����������� menuconfig �˵��ġ����� openwrt ��ϵ����ͨ���� Makefile �ж����������������д����˵��ģ�
       
    target/linux/s3c24xx/Makefile �ļ��������Ŀ��Ŀ¼��
	   
        SUBTARGETS:=dev-s3c2440 dev-s3c2410 dev-fs2410 dev-mini2440 dev-fl2440 dev-gec2410 dev-gec2440 dev-qq2440
   
    2) �� target/linux/s3c24xx �ļ������潨�� dev-fl2440 Ŀ¼���������ص��޸��ļ�����������Ҫ���ļ��� target.mk ����������������ǰ�������õģ�������ʾ��

	    BOARDNAME:=FL2440 Development Board

	    define Target/Description
		    FL2440 Development Board
        endef
       
---------------------------------
##### Copyright 2015 (C) i.lufei([m.lufei@qq.com](mail.qq.com)) #####

����Ҫʹ�ùٷ�������ݣ���ͨ������ٷ����ӽ��л�ȡ��лл������

##### 2015-09-05 #####