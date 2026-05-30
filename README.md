name: Windows RDP with Cloudflare Tunnel

on: 
  workflow_dispatch: # Isse aap manually start kar sakenge

jobs:
  build:
    runs-on: windows-latest
    timeout-minutes: 360 # 6 ghante tak chalega max

    steps:
    - name: Ngrok/Cloudflare Download & Configure
      run: |
        # Cloudflare tunnel download ho rahi hai
        Invoke-WebRequest https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.msi -OutFile cloudflared.msi
        Start-Process msiexec.exe -ArgumentList '/i cloudflared.msi /quiet /norestart' -NoNewWindow -Wait
        
    - name: Enable Windows Remote Desktop (RDP)
      run: |
        # RDP setting enable karne ke liye
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
        
    - name: Create RDP User
      run: |
        # Naya user create karne ke liye (Aap username/password change kar sakte hain)
        net user RDPUser Pass123456 /add
        net localgroup Administrators RDPUser /add
        net localgroup "Remote Desktop Users" RDPUser /add
        
    - name: Start Cloudflare Tunnel & Show Link
      run: |
        # Tunnel start hogi aur iska output 'tunnel.log' mein jayega
        Start-Process -FilePath "C:\Program Files (x86)\cloudflared\cloudflared.exe" -ArgumentList "tunnel --url rdp://localhost:3389" -RedirectStandardError "tunnel.log" -NoNewWindow
        
        # 10 second wait taake tunnel link generate ho jaye
        Start-Sleep -Seconds 10
        
        # Log file se .trycloudflare.com wala link nikal kar screen par dikhana
        $link = Get-Content tunnel.log | Select-String -Pattern "https://.*\.trycloudflare\.com" | ForEach-Object { $_.Matches.Value }
        
        Write-Host "=========================================" -ForegroundColor Green
        Write-Host "YOUR CLOUDFLARE RDP LINK IS BELOW:" -ForegroundColor Cyan
        Write-Host $link -ForegroundColor Yellow
        Write-Host "=========================================" -ForegroundColor Green

    - name: Keep Alive Loop
      run: |
        # Yeh step RDP ko active rakhega
        $loop = $true
        while($loop) {
            Start-Sleep -Seconds 60
        }
