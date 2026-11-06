
name: Windows Tailscale RDP Connection Test

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  secure-rdp:
    runs-on: windows-2022
    timeout-minutes: 30000

    steps:
      - name: Validate OAuth Secrets
        shell: pwsh
        env:
          TS_CLIENT_ID: ${{ kepKJqxNN621CNTRL }}
          TS_CLIENT_SECRET: ${{ tskey-client-kepKJqxNN621CNTRL-TvQk1sYMYt6k9M1rgDzat6TJfgrihfJi8 }}
        run: |
          if ([string]::IsNullOrWhiteSpace($env:TS_CLIENT_ID)) {
            throw "Missing secret: TS_OAUTH_CLIENT_ID"
          }

          if ([string]::IsNullOrWhiteSpace($env:TS_CLIENT_SECRET)) {
            throw "Missing secret: TS_OAUTH_SECRET"
          }

          Write-Host "OAuth credentials are configured."

      - name: Connect GitHub Runner to Tailscale
        uses: tailscale/github-action@v4
        with:
          oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
          oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
          tags: tag:ci
          ping: 100.74.98.81

      - name: Verify Tailscale Status
        shell: pwsh
        run: |
          Write-Host "===== TAILSCALE STATUS ====="
          tailscale status

      - name: Test RDP Port
        shell: pwsh
        run: |
          $TargetIP = "100.74.98.81"
          $Port = 3389

          Write-Host "Testing connection to $TargetIP on port $Port"

          $Result = Test-NetConnection `
            -ComputerName $TargetIP `
            -Port $Port `
            -WarningAction SilentlyContinue

          if (-not $Result.TcpTestSucceeded) {
            throw "RDP connection failed. Check Tailscale access rules and Windows firewall."
          }

          Write-Host "SUCCESS: RDP port 3389 is reachable."

      - name: Finish
        shell: pwsh
        run: |
          Write-Host "Tailscale connectivity test completed."
