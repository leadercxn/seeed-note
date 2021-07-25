# Yocto-devtool


## devtool ������
* devtool �����������
    1. devtool add
        �Զ�����һ�� recipe
        + demo1����ӱ��ص�Դ�ļ�
            ```sh
                $ ls -a ~/code/helloyocto/
                    main.c Makefile
                [build] $ devtool add helloyocto ~/code/helloyocto #add��ʹ�� yocto�����ⲿ�����Ŀ¼
                [build] $ ls workspace/
                [build] $ cat workspace/recipes/helloyocto/helloyocto.bb
            ```
            ������ᴴ�� workspace Ŀ¼����һ��ʼ�����ڣ�
        + demo2: ���github��gieee�ϵ�Դ��
            ```
                [build] $ devtool add --srcbranch develop learnyocto [git-source-url��https://xxxx.git]
                [build] $ ls workspace/source/learnyocto
            ```
    2. devtool modify
        ��ָ�����е�recipe��ȷ���Ӻδ���ȡԴ�����Լ���ζ�������޲�
    3. devtool upgrade
        �������е�recipe
    4. devtool build
        ���� recipe�� ������bitbake��ʹ��devtool build ����ʱ������ʹ��recipe�ĸ����ƣ�����Ϊ helloyocto ��
        ```
            [build] $ devtool build helloyocto
            [build] $ ls workspace/sources/helloyocto/oe-workdir/image/usr/bin/helloyocto
        ```
    5. devtool reset
        ��workspaceĿ¼��ɾ��ָ����recipe��
    6. devtool status
        �鿴workspace������Щ��Ŀ���Լ����Ե�Դ��·��
    7. devtool build-image
        ����image,���� workspace �е�recipes������ image �ļ�ϵͳ�С������ڿ������޸���֤����;ʹ�ã�����ɹ��ˣ��ͼ��ϵ�ĳ��metadata layerĿ¼�С�
    8. devtool finish
        ���� workspace Ŀ¼�п����� recipe ���ɵ���ʽ�� meta-layer��ȥ
        ```
            devtool finish learnyocto meta-mylayer
        ```
        ���绹Ҫ����ִ���ļ��ŵ����ɵ��ļ�ϵͳ��ȥ�������޸Ķ�Ӧ��.bb�ļ����޸� `IMAGE_INSTALL += xxx` ����ӣ�������Ҫ source xxx_env 
        ```sh
            bitbake core-image-sato -c cleanall
            bitbake core-image-sato

            # ȷ���Ƿ���ӳɹ�
            build/tmp-qemux86-64/work/qemux86_64-poky-linux/core-image-sato/1.0-r0/rootfs/usr/bin/learnyocto 
        ```
        �鿴��װ����嵥Ҳ����ȷ��
        ```sh
            cat build/tmp/deploy/images/xxxx/xxxxx.manifest
        ```
    9. devtool deploy-target
        �� recipe�Ĺ����� do_install �����а�װ�������ļ�ֱ���������`�����ŵ�Ŀ�����`�ϣ�Ŀ�����������Ҫ����ssh����
            ��������            Ŀ�����
                |                   |
                |                   |
                | ----------------> |
        ```sh
            [build] $  devtool deploy-target learnyocto  root@192.168.1.101:/   # ��Ŀ�겻���Զ���������������Ŀ�ͳ���
            [Ŀ���豸] $ ./usr/bin/learnyocto
        ```

        + ����ʧ��ԭ��
            + ��Ҫ����ĳ����������У����߿ⱻ����ʹ����
            + Ŀ�������û�а�װӦ�ó����������Ŀ������������
    10. devtool undeploy-target
        ������ devtool deploy-target �������ƣ�������ǰ�沿�����ڻ����ϵĿ�ִ�г���ж�أ��Ƴ���
        ```sh
            devtool undeploy-target learnyocto root@192.168.1.101:/ # ��������˶��Ӧ�ó��򣬿���ʹ�� -a ѡ�����ȫ��ɾ�����Ӷ���Ŀ���豸�ָ�����ԭʼ״̬
        ```
    11. 
        
 
* �������˵��
    1. 
        ```sh
            devtool -h
            devtool add --help
        ```

## devtool �Ĺ���Ŀ¼�ܹ�
* devtool ʹ��һ�� workspace������ɹ���������������ʹ�õĹ�����������
    + Ŀ¼�ܹ�
        build
            |- workspace-layer
                |- attic
                |-appends
                |-conf
                |-recipes
                |-sources




