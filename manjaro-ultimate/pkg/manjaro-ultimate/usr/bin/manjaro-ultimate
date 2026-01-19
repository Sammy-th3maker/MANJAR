#!/usr/bin/env bash
# Manjaro Ultimate Gaming & Repair Tool
# by Сэмми coder ChatGPT

set -euo pipefail
[[ $EUID -ne 0 ]] && { echo "Run with sudo"; exit 1; }

GREEN="\e[32m"; RESET="\e[0m"
PROFILE_DIR="/etc/manjaro-ultimate/profiles"
mkdir -p "$PROFILE_DIR"

CURRENT_POWER="balanced"
GAMING_ENABLED="false"
CURRENT_KERNEL=""
CURRENT_GPU=""

# ---------- UI ----------
logo() {
  clear
  echo -e "${GREEN}"
  cat << "EOF"
███╗   ███╗ █████╗ ███╗   ██╗     ██╗ █████╗ ██████╗
████╗ ████║██╔══██╗████╗  ██║     ██║██╔══██╗██╔══██╗
██╔████╔██║███████║██╔██╗ ██║     ██║███████║██████╔╝
██║╚██╔╝██║██╔══██║██║╚██╗██║██   ██║██╔══██║██╔══██╗
██║ ╚═╝ ██║██║  ██║██║ ╚████║╚█████╔╝██║  ██║██║  ██║
╚═╝     ╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝ ╚════╝ ╚═╝  ╚═╝╚═╝  ╚═╝
EOF
  echo "by Сэмми coder ChatGPT"
  echo -e "${RESET}"
}
pause(){ read -rp "Press ENTER to continue..."; }

# ---------- DETECT ----------
is_laptop(){ [[ -d /sys/class/power_supply/BAT0 || -d /sys/class/power_supply/BAT1 ]]; }
cpu_vendor(){ lscpu | awk -F: '/Vendor ID/ {print $2}' | xargs; }
gpu_vendor(){ lspci | grep -E "VGA|3D" | grep -Eo "AMD|NVIDIA|Intel" | head -n1; }
best_governor(){
  local g
  g=$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors 2>/dev/null || true)
  for x in performance schedutil ondemand powersave; do [[ "$g" == *"$x"* ]] && { echo "$x"; return; }; done
  echo performance
}

# ---------- SYSTEM ----------
system_update(){ pacman -Syu --noconfirm; }
fix_packages(){ pacman -Dk; pacman -Syyu --noconfirm; }
clean_cache(){ pacman -Sc --noconfirm; }

# ---------- DRIVERS ----------
install_amd(){ CURRENT_GPU="AMD"; pacman -S --needed mesa vulkan-radeon lib32-vulkan-radeon xf86-video-amdgpu --noconfirm; }
install_intel(){ CURRENT_GPU="Intel"; pacman -S --needed mesa vulkan-intel lib32-vulkan-intel --noconfirm; }
install_nvidia(){ CURRENT_GPU="NVIDIA"; mhwd -a pci nonfree 0300; }

# ---------- GPU TUNE ----------
gpu_tuning(){
  case "$(gpu_vendor)" in
    AMD) echo high > /sys/class/drm/card0/device/power_dpm_force_performance_level 2>/dev/null || true ;;
    NVIDIA) nvidia-smi -pm 1 2>/dev/null || true ;;
    Intel) echo performance | tee /sys/class/drm/card*/device/power_profile 2>/dev/null || true ;;
  esac
}

# ---------- GAMING ----------
install_gaming(){
  pacman -S --needed steam lutris gamemode lib32-gamemode mangohud cpupower --noconfirm
  systemctl enable --now gamemoded
}
gaming_boost(){
  GAMING_ENABLED="true"
  cpupower frequency-set -g performance || true
  sysctl -w vm.swappiness=10 >/dev/null
  case "$(cpu_vendor)" in
    GenuineIntel) echo 0 > /sys/devices/system/cpu/intel_pstate/no_turbo 2>/dev/null || true ;;
    AuthenticAMD) echo performance | tee /sys/devices/system/cpu/cpufreq/policy*/energy_performance_preference >/dev/null 2>&1 || true ;;
  esac
  gpu_tuning
}

# ---------- POWER ----------
power_powersave(){
  CURRENT_POWER="powersave"
  is_laptop && pacman -S --needed tlp --noconfirm && systemctl enable --now tlp
  cpupower frequency-set -g powersave || true
}
power_balanced(){
  CURRENT_POWER="balanced"
  is_laptop && pacman -S --needed tlp --noconfirm && systemctl enable --now tlp
  cpupower frequency-set -g "$(best_governor)" || true
}
power_performance(){
  CURRENT_POWER="performance"
  systemctl stop tlp 2>/dev/null || true
  systemctl disable tlp 2>/dev/null || true
  cpupower frequency-set -g performance || true
}

# ---------- KERNEL ----------
install_kernel(){ CURRENT_KERNEL="$1"; mhwd-kernel -i "$1" rmc; }

# ---------- PROFILES ----------
save_profile(){
  read -rp "Profile name: " n
  cat > "$PROFILE_DIR/$n.conf" <<EOF
POWER=$CURRENT_POWER
GAMING=$GAMING_ENABLED
KERNEL=$CURRENT_KERNEL
GPU=$CURRENT_GPU
EOF
  echo "Saved profile: $n"
}
load_profile(){
  read -rp "Profile name: " n
  source "$PROFILE_DIR/$n.conf"
  case "$POWER" in
    powersave) power_powersave ;;
    balanced) power_balanced ;;
    performance) power_performance ;;
  esac
  [[ "$GAMING" == "true" ]] && gaming_boost
  [[ -n "$KERNEL" ]] && install_kernel "$KERNEL"
  case "$GPU" in
    AMD) install_amd ;;
    Intel) install_intel ;;
    NVIDIA) install_nvidia ;;
  esac
}

# ---------- MENUS ----------
menu(){
  while true; do
    logo
    echo "1) System"
    echo "2) Drivers"
    echo "3) Gaming"
    echo "4) Power"
    echo "5) Kernels"
    echo "6) Profiles"
    echo "0) Exit"
    read -rp "> " c
    case "$c" in
      1) system_update ;;
      2) drivers_menu ;;
      3) gaming_menu ;;
      4) power_menu ;;
      5) kernel_menu ;;
      6) profile_menu ;;
      0) exit ;;
    esac
    pause
  done
}

drivers_menu(){ logo; echo "1) AMD 2) Intel 3) NVIDIA"; read -rp "> " d; [[ $d == 1 ]] && install_amd; [[ $d == 2 ]] && install_intel; [[ $d == 3 ]] && install_nvidia; }
gaming_menu(){ logo; echo "1) Install Gaming 2) Gaming Boost"; read -rp "> " g; [[ $g == 1 ]] && install_gaming; [[ $g == 2 ]] && gaming_boost; }
power_menu(){ logo; echo "1) Powersave 2) Balanced 3) Performance"; read -rp "> " p; [[ $p == 1 ]] && power_powersave; [[ $p == 2 ]] && power_balanced; [[ $p == 3 ]] && power_performance; }
kernel_menu(){ logo; echo "1) linux 2) lts 3) zen"; read -rp "> " k; [[ $k == 1 ]] && install_kernel linux; [[ $k == 2 ]] && install_kernel linux-lts; [[ $k == 3 ]] && install_kernel linux-zen; }
profile_menu(){ logo; echo "1) Save 2) Load"; read -rp "> " p; [[ $p == 1 ]] && save_profile; [[ $p == 2 ]] && load_profile; }

menu
