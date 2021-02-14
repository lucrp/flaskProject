# Python Project

## Introduction
Ce projet écrit en Flask récupère les informations système d'un agent, les sauvegarde dans une Base de Données SQLite et les affiche dans une interface web.

Le projet n'est pas encore fini. Il reste la partie pour récupérer les informations dans la Base de Données et les afficher dans une route.

## Informations système
Les informations système récupéres : 
- architecture 
- machine
- uptime
- hostname 
- system
- distro_host
- kernel
- processor

Ces informations sont ensuite ajoutées dans un dictionnaire. Code :
```python
def uptime():
    try:
        f = open("/proc/uptime")
        contents = f.read().split()
        f.close()
    except:
        return "Cannot open uptime file: /proc/uptime"

    total_seconds = float(contents[0])

    #  print(total_seconds)

    # Helper vars:
    MINUTE = 60
    HOUR = MINUTE * 60
    DAY = HOUR * 24

    # Get the days, hours, etc:
    days = int(total_seconds / DAY)
    hours = int((total_seconds % DAY) / HOUR)
    minutes = int((total_seconds % HOUR) / MINUTE)
    seconds = int(total_seconds % MINUTE)

    # Build up the pretty string (like this: "N days, N hours, N minutes, N seconds")
    string = ""
    if days > 0:
        string += str(days) + " " + (days == 1 and "day" or "days") + ", "
    string += str(hours) + ":"
    string += str(minutes) + ":"
    string += str(seconds)

    return string


# CPU(s)
def get_processor_name():
    if platform.system() == "Windows":
        return platform.processor()
    elif platform.system() == "Darwin":
        os.environ['PATH'] = os.environ['PATH'] + os.pathsep + '/usr/sbin'
        command = "sysctl -n machdep.cpu.brand_string"
        return subprocess.check_output(command).strip()
    elif platform.system() == "Linux":
        command = "cat /proc/cpuinfo"
        all_info = subprocess.check_output(command, shell=True).strip()
        for line in all_info.decode().split("\n"):
            if "model name" in line:
                return re.sub(".*model name.*:", "", line, 1)
    return ""


# Distribution
dist = distro.linux_distribution(full_distribution_name=False)
distInfo = dist[0] + " " + dist[1] + " " + dist[2]

data = {"architecture": platform.architecture()[0], "machine": platform.machine(), "uptime_host": uptime(),
        "hostname": platform.uname()[1], "system": platform.system(), "distro_host": distInfo,
        "kernel": platform.release(), "processor": get_processor_name()}
```
