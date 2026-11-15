HELM uses multiple communication methods to exchange information between the handheld device and the PC companion.

Different communication metods are selected based on the requirements ofd each data category.

Communication Architechure
    PC telemetry
        PC -> Pi
        TCP
        Transfer system performace measurements
    File Transfer
        Bidirectional
        TCP
        Reliable file transfer
    Controller Input
        Pi -> PC
        UDP (At least for Joysticks, button input tbd)
        Low-latency controller updates 
    Connection monitoring
        Bidirectional
        TBD
        Monitor communication health
    Desktop Video Stream
        PC -> Pi
        UDP-based
        Transmit live desktop imagery
    Mouse and keyboard input
        Pi -> PC
        Done through USB, packets through TCP
        Physical input through dedicated USB interface

TCP Communication
    Used for data where reliable and ordered delivery is important
    Telemetry
        PC performance transmitted from Windows companion to Pi
        Specifications:
            Data format: JSON
            Character encoding: UTF-8
            Transport: TCP
            Direction: PC -> Pi
            Message framing:

        Telemetry includes:
            CPU, RAM, and GPU usage
            Remaining PC storage (Both C: and D: drives)
            CPU and GPU temps
    File Transfer
        Message structure:
            TBD
        Handling:
            TBD

UDP Communication
    Used for time-sensitive information where minimizing latency is more important than the indivdual packets sent
    Controller input:
        Message structure:
            TBD
        Handling:
            TBD
    Desktop Streaming
        Capture entire selected windows monitor
        Switch between monitors
        Preserve aspect ratio while scaling to 1024x600 display
        Support monitors with different aspect ratios/resolutions
        Streaming on demand
        Will use UDP-based transmission to prioritize low-latency delivery
        VERY TBD
    Connection Health Monitoring
        Connection check interval: 2s
        Response timeout: 1.5s
        Unstable threshold: 1 missed check
        Disconnection threshold: 3 missed checks
        Recovery threshold: 1 successful response
        Packet-loss measurement window: 60s
        Connection states:
            Connected: Most recent check had a valid response
            Unstable: 1-2 checks missed
            Disconnected: 3+ checks missed
        Packet loss and latency are tracked independently of connection state

USB communication:
    Mouse and keyboard input handeled through a USB interface, though the pi has an onscreen keyboard and touchscreen
    Input requirements:
        Mouse movement and button input
        Keyboard input
        Communication with PC through USB
        Independence from video streaming
    Protocols:
        TBD

Protocol Design Considerations:
    Moving forward, these considerations will guide development
        Message identification/type fields
        Protocol versioning
        Timestamp conventions
        Error handling
        Device authenticaition
        Sequence numbering for UDP packets
        Handling outdated or duplicate data
        Recovery from an interrrupted connection
        Resource/Bandwith contraints

