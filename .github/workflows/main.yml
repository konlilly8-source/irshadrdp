name: Cloudflare RDP Server

on: 
  workflow_dispatch: # Taake aap manually click karke on kar sako

jobs:
  build:
    runs-on: windows-latest
    timeout-minutes: 360

    steps:
    - name: Download Cloudflare Tunnel (cloudflared)
      run: |
        # Cloudflare binary ko sidha download kar ke 'cf.exe' ka naam de rahe hain
        Invoke-WebRequest https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe -OutFile cf.exe
        
    - name: Enable Windows RDP Settings
      run: |
        # Windows RDP ko allow aur enable karne ke liye
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
        
    - name: Setup RDP Login Credentials
      run: |
        # Username: RDPUser | Password: Pass123456
        net user RDPUser Pass123456 /add
        net localgroup Administrators RDPUser /add
        net localgroup "Remote Desktop Users" RDPUser /add
        
    - name: Start Cloudflare Tunnel & Extract Link
      run: |
        # Tunnel start karke output log file mein save ho rahi hai
        Start-Process -FilePath ".\cf.exe" -ArgumentList "tunnel --url rdp://localhost:3389" -RedirectStandardError "tunnel.log" -NoNewWindow
        
        # 10 second wait taake link generate ho jaye
        Start-Sleep -Seconds 10
        
        # Log file se exact trycloudflare ka link nikal kar screen par print karna
        $link = Get-Content tunnel.log | Select-String -Pattern "https://.*\.trycloudflare\.com" | ForEach-Object { $_.Matches.Value }
        
        Write-Host "=========================================" -ForegroundColor Green
        Write-Host "AAPKA CLOUDFLARE LINK YEH HAI:" -ForegroundColor Cyan
        Write-Host $link -ForegroundColor Yellow
        Write-Host "=========================================" -ForegroundColor Green

    - name: Keep RDP Alive Loop
      run: |
        # RDP ko active rakhne ke liye loop
        $loop = $true
        while($loop) {
            Start-Sleep -Seconds 60
        }
