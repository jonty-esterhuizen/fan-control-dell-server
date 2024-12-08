# Dynamic Fan Speed Control for Servers Using `ipmitool`

This PowerShell script dynamically adjusts server fan speeds based on real-time temperature readings. It supports multiple servers, fine-grained temperature ranges, and custom fan speeds for optimal thermal management.

---

## Prerequisites

1. **Install `ipmitool`**:
   - Ensure `ipmitool` is installed and available in your system's `PATH`.
   - For Windows, download `ipmitool` binaries and add the path to your environment variables.

2. **Enable IPMI Over LAN**:
   - Ensure that IPMI is enabled on your servers. Use the following RACADM command if necessary:
     ```
     racadm set idrac.ipmi.lanenable Enabled
     ```

3. **Server Credentials**:
   - Ensure you have valid IPMI credentials for each server.

4. **PowerShell**:
   - Use PowerShell 5.1 or later for running the script.

---

## Features

- **Multi-Server Support**: Supports dynamic fan speed control for multiple servers.
- **Customizable Thresholds**: Adjust temperature ranges and fan speeds as per your requirements.
- **Real-Time Monitoring**: Reads sensor data to dynamically adjust fan speeds.
- **Error Handling**: Captures and logs errors during execution.

---

## How It Works

1. The script reads real-time temperature data from the servers using `ipmitool`.
2. Based on pre-defined temperature thresholds, the script adjusts the fan speed dynamically.
3. Supports 10 temperature ranges with corresponding fan speeds.

---

## Usage

### 1. Clone or Download the Script
Save the script as `AdjustFanSpeeds.ps1`.

### 2. Modify Server and Threshold Settings
Customize the `$Servers` array and `$Thresholds` hashtable in the script:

#### Example Server Configuration:
```
[ 
    @{ IP = "192.168.0.41"; Username = "root"; Password = "calvin" },
    @{ IP = "192.168.0.120"; Username = "root"; Password = "calvin" }
]
```

#### Example Temperature Thresholds:
```
[hashtable]$Thresholds = @{
    Range1  = @{ Temp = 20; FanSpeed = "0x0A" }  # Below 20°C -> 10%
    Range2  = @{ Temp = 30; FanSpeed = "0x14" }  # 20°C to 30°C -> 20%
    Range3  = @{ Temp = 40; FanSpeed = "0x1E" }  # 30°C to 40°C -> 30%
    Range4  = @{ Temp = 50; FanSpeed = "0x28" }  # 40°C to 50°C -> 40%
    Range5  = @{ Temp = 60; FanSpeed = "0x32" }  # 50°C to 60°C -> 50%
    Range6  = @{ Temp = 70; FanSpeed = "0x3C" }  # 60°C to 70°C -> 60%
    Range7  = @{ Temp = 80; FanSpeed = "0x46" }  # 70°C to 80°C -> 70%
    Range8  = @{ Temp = 90; FanSpeed = "0x50" }  # 80°C to 90°C -> 80%
    Range9  = @{ Temp = 100; FanSpeed = "0x5A" } # 90°C to 100°C -> 90%
    Range10 = @{ FanSpeed = "0x64" }             # Above 100°C -> 100%
}
```

---

### 3. Run the Script
Run the script using PowerShell:
```
.\AdjustFanSpeeds.ps1
```

---

## End User License Agreement (EULA)

### Disclaimer of Warranty
This script is provided "as is," without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. The entire risk as to the quality and performance of the script is with you.

### Limitation of Liability
In no event shall the author or contributors be liable for any damages (including, but not limited to, procurement of substitute goods or services, loss of use, data, or profits, or business interruption) arising in any way out of the use of this script, even if advised of the possibility of such damage.

### Acceptance of Terms
By using this script, you acknowledge and agree to the terms outlined above. If you do not agree to these terms, do not use the script.

---

## Script Code

```
param (
    [array]$Servers = @(
        @{ IP = "192.168.0.41"; Username = "root"; Password = "calvin" },
        @{ IP = "192.168.0.120"; Username = "root"; Password = "calvin" }
    ),
    [hashtable]$Thresholds = @{
        Range1  = @{ Temp = 20; FanSpeed = "0x0A" }  # Below 20°C -> 10%
        Range2  = @{ Temp = 30; FanSpeed = "0x14" }  # 20°C to 30°C -> 20%
        Range3  = @{ Temp = 40; FanSpeed = "0x1E" }  # 30°C to 40°C -> 30%
        Range4  = @{ Temp = 50; FanSpeed = "0x28" }  # 40°C to 50°C -> 40%
        Range5  = @{ Temp = 60; FanSpeed = "0x32" }  # 50°C to 60°C -> 50%
        Range6  = @{ Temp = 70; FanSpeed = "0x3C" }  # 60°C to 70°C -> 60%
        Range7  = @{ Temp = 80; FanSpeed = "0x46" }  # 70°C to 80°C -> 70%
        Range8  = @{ Temp = 90; FanSpeed = "0x50" }  # 80°C to 90°C -> 80%
        Range9  = @{ Temp = 100; FanSpeed = "0x5A" } # 90°C to 100°C -> 90%
        Range10 = @{ FanSpeed = "0x64" }             # Above 100°C -> 100%
    }
)

# Function to adjust fan speed based on temperature
function Adjust-FanSpeed {
    param (
        [string]$ServerIP,
        [string]$Username,
        [string]$Password,
        [hashtable]$Thresholds
    )

    try {
        # Fetch temperatures
        $temps = & ipmitool -I lanplus -H $ServerIP -U $Username -P $Password sensor | Select-String "Temp" | ForEach-Object { ($_ -split '\|')[1] -as [float] }

        # Determine the appropriate fan speed
        $fanSpeed = $Thresholds.Range10.FanSpeed  # Default to the highest range
        foreach ($key in $Thresholds.Keys) {
            if ($Thresholds[$key].Temp -and $temps -lt $Thresholds[$key].Temp) {
                $fanSpeed = $Thresholds[$key].FanSpeed
                break
            }
        }

        # Apply the fan speed
        & ipmitool -I lanplus -H $ServerIP -U $Username -P $Password raw 0x30 0x30 0x02 0xff $fanSpeed
        Write-Output "[$ServerIP] Fan speed set to $fanSpeed (Temp: $temps °C)"
    } catch {
        Write-Output "[$ServerIP] Error adjusting fan speed: $_"
    }
}

# Process each server
foreach ($server in $Servers) {
    Adjust-FanSpeed -ServerIP $server.IP -Username $server.Username -Password $server.Password -Thresholds $Thresholds
}
```

---

## Contributing

Feel free to submit issues, fork the repository, or make pull requests to enhance the script.

---

## License

This project is licensed under the MIT License.
---

## End User License Agreement (EULA)

### Disclaimer of Warranty
This script is provided "as is," without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. The entire risk as to the quality and performance of the script is with you.

### Limitation of Liability
In no event shall the author or contributors be liable for any damages (including, but not limited to, procurement of substitute goods or services, loss of use, data, or profits, or business interruption) arising in any way out of the use of this script, even if advised of the possibility of such damage.

### Acceptance of Terms
By using this script, you acknowledge and agree to the terms outlined above. If you do not agree to these terms, do not use the script.

---
