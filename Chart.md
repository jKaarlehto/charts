sequenceDiagram title Process Sequence Diagram
    actor User
    participant P0
    participant C0
    participant C1
    participant SharedMemory
    participant P1

    User ->> P0: Start program
    activate User
    activate P0
    P0 -->> P0: fildes[] = os.pipe()
    P0 -->> C0: pid = os.fork()
    activate C0

    
    P0 ->> User: input_str = input()
    User ->> P0: Input string
    deactivate User
    

    critical  input string via pipe
    P0 -->> P0: os.write(fildes[1], input_str.encode())
        
        P0 --> C0: pipe
        Note right of P0: C0 will be blocked<br> until data is written<br>to pipe
        C0 -->> C0: os.read(fildes[0])
    end

    par P0 waits for C0 to terminate
        P0 --> C0: os.waitpid(pid)
        deactivate P0
        and
        C0 -->> C0: parse string
        Note right of C0: create shared memory
        C0 --> SharedMemory: shm = SharedMemory(create=True, size=len(data))
        activate SharedMemory
        C0 ->> SharedMemory: parsed string
        C0 -->> C1: pid2 = os.fork()
        activate C1
        par C0 waits for P1/C1 to terminate
        C0 --> P1: os.waitpid(pid2)
        deactivate C0
        and
        Note right of C1: temporary C1 becomes<br> a separate process P1
        C1 -->> P1: os.ececl(./P1)
        C1 ->> P1: shm.name (argument)
        activate P1
        deactivate C1
        Note left of P1: attach to shared memory
        P1 -->> SharedMemory: shm = SharedMemory(create=False, name=shm.name)
        SharedMemory ->> P1: parsed string
        P1 -->> P1: find missing alphabet
        P1 ->> User: print output
        P1 -->> C0: exit(0)
        deactivate P1
        end
        activate C0
        C0 --> C0: waitpid(pid2)
        C0 -->> SharedMemory: shm.unlink()
        deactivate SharedMemory
        C0 -->> P0: exit(0)
        deactivate C0
    end
    activate P0
    P0 -->> P0: waitpid(pid)
    P0 -->> User: exit(0)
    deactivate P0
    
    

