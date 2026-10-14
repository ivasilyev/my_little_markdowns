# Synthesia tweaks

## Disable network access

```shell script
# List of paths using PowerShell environment variables
$apps = @(
    "${env:ProgramFiles(x86)}\Synthesia\Synthesia.exe",
    "${env:AppData}\Synthesia\Synthesia.exe"
)

# Loop through each application
foreach ($app in ${apps} ) {
    # Verify the file exists before making firewall rules
    if (Test-Path "${app}") {
        $filename = Split-Path -Leaf "${app}"
        Write-Host "Blocking network access for: ${app}" -ForegroundColor Cyan

        # Block Inbound Traffic
        New-NetFirewallRule -DisplayName "Block Inbound - ${filename}" `
                            -Direction Inbound `
                            -Program ${app} `
                            -Action Block `
                            -Description "Automated inbound block rule for ${filename}" `
                            -Enabled True

        # Block Outbound Traffic
        New-NetFirewallRule -DisplayName "Block Outbound - ${filename}" `
                            -Direction Outbound `
                            -Program ${app} `
                            -Action Block `
                            -Description "Automated outbound block rule for ${filename}" `
                            -Enabled True
    } else {
        Write-Warning "File not found, skipping: ${app}"
    }
}
```
