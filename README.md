import os
import time
import shutil
import platform
import datetime
import json

# Thresholds for Data Center Server Alerts
CPU_TEMP_THRESHOLD_C = 80.0
DISK_USAGE_THRESHOLD_PCT = 85.0
LOG_FILE = "server_telemetry_alerts.json"

def get_cpu_temperature():
    """Reads CPU thermal sensor data on Linux systems via /sys/class/thermal."""
    if platform.system().lower() == "linux":
        try:
            with open("/sys/class/thermal/thermal_zone0/temp", "r") as f:
                temp_millidegrees = int(f.read().strip())
                return round(temp_millidegrees / 1000.0, 2)
        except FileNotFoundError:
            return None
    return None

def get_disk_health(path="/"):
    """Checks total, used, and free storage capacity and calculates utilization percentage."""
    total, used, free = shutil.disk_usage(path)
    usage_pct = round((used / total) * 100, 2)
    return {
        "total_gb": round(total / (1024**3), 2),
        "used_gb": round(used / (1024**3), 2),
        "free_gb": round(free / (1024**3), 2),
        "usage_pct": usage_pct
    }

def get_system_load():
    """Fetches 1m, 5m, and 15m load averages on Unix/Linux servers."""
    if hasattr(os, "getloadavg"):
        load1, load5, load15 = os.getloadavg()
        return {"1m": round(load1, 2), "5m": round(load5, 2), "15m": round(load15, 2)}
    return {"1m": "N/A", "5m": "N/A", "15m": "N/A"}

def collect_telemetry():
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    cpu_temp = get_cpu_temperature()
    disk = get_disk_health()
    load = get_system_load()

    alerts = []
    if cpu_temp and cpu_temp > CPU_TEMP_THRESHOLD_C:
        alerts.append(f"HIGH THERMAL WARNING: CPU Temp at {cpu_temp}°C (Threshold: {CPU_TEMP_THRESHOLD_C}°C)")
    if disk["usage_pct"] > DISK_USAGE_THRESHOLD_PCT:
        alerts.append(f"STORAGE CAPACITY ALERT: Disk usage at {disk['usage_pct']}%")

    telemetry_record = {
        "timestamp": timestamp,
        "hostname": platform.node(),
        "os": platform.system(),
        "cpu_temp_c": cpu_temp,
        "load_average": load,
        "disk_telemetry": disk,
        "status": "CRITICAL" if alerts else "HEALTHY",
        "alerts": alerts
    }

    print(json.dumps(telemetry_record, indent=2))
    if alerts:
        with open(LOG_FILE, "a") as f:
            f.write(json.dumps(telemetry_record) + "\n")

if __name__ == "__main__":
    print("Initializing Server Hardware & Thermal Telemetry Monitor...")
    collect_telemetry()
