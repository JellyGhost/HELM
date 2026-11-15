The telemetry data we are monitoring is:
    CPU, RAM, and GPU usage
        All in %, with RAM in bot % and MB
    CPU and GPU temps
    Storage
        By default the C: and D: drives

We are sampling all of the above at a rate of once per 15s.

We will also check the connection (ping, packet loss, connectivity status) once per 2s
    Ping measured in ms
    Packet loss in %, measured as unanswered probes/total probes sent. checked over a 60s rolling window to prevent sudden jumps
    Connectivity status as Connected, Unstable, and Disconnected
        Connected:
            0 missed checks
            Most recent probe recieved valid response
            PC companion is communicating normally
        Unstable:
            1-2 consecutive missed checks
            Communication is intermittent or a response is delayed beyond timeout
            Display the most recent known information sent by the PC
        Disconnected:
            3 or more consecutively missed checks
            No valid response within the defined timeout
            Stop treating incoming PC info as current
    Response timeout will be 1.5s by default