  
name: ViduRDPCI
#gtfjjyhjyukuk
on: workflow_dispatch

jobs:
  build:

    runs-on: windows-latest
    timeout-minutes: 9999

    steps:
    - name: Download Ngrok & NSSM
      run: |
        Invoke-WebRequest https://github.com/beratby486/WindowsRDP/raw/main/ngrok.exe -OutFile ngrok.exe
        Invoke-WebRequest https://github.com/extizi/WindowsRDP/raw/main/nssm.exe -OutFile nssm.exe
    - name: Copy NSSM & Ngrok to Windows Directory.
      run: | 
        copy nssm.exe C:\Windows\System32
        copy ngrok.exe C:\Windows\System32
    - name: Connect your NGROK account
      run: .\ngrok.exe authtoken $Env:NGROK_AUTH
      env:
        NGROK_AUTH: ${{ secrets.NGROK_AUTH }}
    - name: Download Important Files.
      run: |
        Invoke-WebRequest https://github.com/extizi/WindowsRDP/raw/main/NGROK-AP.bat -OutFile NGROK-AP.bat
        Invoke-WebRequest https://github.com/extizi/WindowsRDP/raw/main/NGROK-CHECK.bat -OutFile NGROK-CHECK.bat
        Invoke-WebRequest https://github.com/extizi/WindowsRDP/raw/main/loop.bat -OutFile loop.bat
    - name: Make YML file for NGROK.
      run: start NGROK-AP.bat
    - name: Enable RDP Access.
      run: | 
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
    - name: Tunnel
      run: sc start ngrok
    - name: Connect to your RDP 2core-7GB Ram.
      run: cmd /c NGROK-CHECK.bat
    - name: Finish
      run: cmd /c loop.bat 
