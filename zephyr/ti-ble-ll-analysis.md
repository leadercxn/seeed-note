# ti-ble layer 分析
1. 
    In BLE, there are 5 states a device can be in.
        Standby
        Advertising
        Scanning
        Initiating
        Connected
2. 
    There are six events that scheduler needs to take care of. These are:
        Initiating
        Advertising
        Scanning
        Periodic Advertising
        Periodic Scanning
        Connection

3. 
    The link layer executes these tasks in a pre-defined order: Initiating –> Advertising –> Scanning –> Periodic Advertising –> Periodic Scanning. Each task will get its time and will be executed eventually.

