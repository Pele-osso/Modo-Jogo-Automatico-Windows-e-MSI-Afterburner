# Modo_Jogo_Automatico.bat (Hardware-Aware + 4 Perfis UV/OC + AMD FSR + SDXL + OBS & Realtek)

> **Nota de Autoria:** Este projeto e todo o código do script `.bat` foram criados automaticamente pelo **Antigravity** (assistente de IA da Google DeepMind), de forma 100% dinâmica e portátil para qualquer configuração de PC (Intel/AMD/NVIDIA, Desktop ou Notebook), sem dados pessoais.

---

## O que este `.bat` faz automaticamente (adaptado às peças do seu PC)

### 1. 4 Perfis de GPU no MSI Afterburner (Undervolt + Overclock dinâmicos e seguros)
O script preserva seu **`Slot 1` (`[Profile1]`) intacto como backup** (e grava `[Startup]` sempre no nível seguro **Leve** para evitar instabilidade na inicialização), calibrando automaticamente **4 curvas Voltage/Frequency (`VFCurve`) + Overclock de VRAM + Power Limit** nos Slots `2` a `5` sobre a tabela `[Defaults]` da sua GPU e tipo de sistema (Desktop/Notebook):

* **`Slot 2` — Desempenho Leve (`-leve`):** *Undervolt* + *Overclock Leve* com margem de estabilidade ampliada. Ideal para uso diário gamer com temperatura baixa e silêncio.
* **`Slot 3` — Desempenho Médio (`-medio`):** *Undervolt* + *Overclock Médio* balanceado (~1.5x Core / ~1.66x VRAM sobre o perfil Leve), com teto de tensão otimizado. Ideal para jogos competitivos, streaming ou edição.
* **`Slot 4` — Desempenho Boost (`-boost` | Padrão Principal):** *Undervolt* pronunciado + *Overclock Boost Seguro* mantendo o teto de tensão bem abaixo do limite *stock* de fábrica (evitando estrangulamento térmico/TDP, *BusReset TDR* PCIe em GPUs alimentadas pelo slot e travamentos sob carga pesada). Ideal como predefinição principal para jogos pesados, **Jogo + Live/Gravação simultâneos** ou **renderização / IA (SDXL)**.
* **`Slot 5` — Economia (`-economia`):** *Undervolt* de Eficiência + *Power Limit* reduzido, derrubando consumo e temperatura na área de trabalho quando nenhum jogo ou processo pesado está aberto. *(Em Desktop, força `PCIe ASPM = Desligado` em todos os planos de energia para evitar desconexões do barramento da GPU ao alternar para Economia).*

---

### 2. AMD FSR (1/2/3), Ponte DLSS -> FSR & Upscaling Universal (`-fsr`, `-instalarfsr`, `[12]`)
* **Detecção de Arquitetura de GPU:** Verifica automaticamente suporte a *Tensor Cores* e quantidade de VRAM; placas sem hardware DLSS recebem prioridade automática para **AMD FSR 1/2/3**, **OptiScaler** e **Magpie** sem tentar forçar recursos incompatíveis.
* **Calibração Direta em Jogos:**
  * **Counter-Strike 2 (`cs2_video.txt`):** Ativa `setting.videocfg_fsr_detail` adaptado à resolução/VRAM (`1` = Ultra Quality em Low-VRAM <= 1440p), habilita baixa latência (`r_low_latency = 2` NVIDIA Reflex Boost / `1` AMD), remove `mat_vsync` (`0`) e reduz pressão de VRAM (`gpu_mem_level = 0` quando necessário).
  * **Overwatch (`Settings_v0.ini`):** Remove travas de VSync/Triple Buffering (`0`), sincroniza `DesiredFrameRate` com a taxa Hz real do monitor, habilita `HighTickInput = "1"` e ajusta `RenderScale` / `AADetail` em GPUs de `<= 4 GB` VRAM (sempre salvando `.bak_modojogo` antes de editar).
* **Instalador Integrado de Magpie & OptiScaler (opção `[12]` / `-instalarfsr`):**
  * Baixa do GitHub oficial o **Magpie** (aplica **AMD FSR 1.0 universal** em qualquer jogo em modo janela/sem bordas via `Win + Shift + A` e cria o atalho `Ativar_FSR_Universal_Magpie.bat` na Área de Trabalho).
  * Baixa o **OptiScaler** (converte chamadas DLSS/XeSS do jogo para **AMD FSR 2.x/3.x**), procura automaticamente jogos com `nvngx_dlss.dll` instalados nos discos e oferece instalação segura com backup `.bak_modojogo`.
  * Também reconhece **Lossless Scaling** se já estiver instalado pela Steam e prioriza seus processos durante os jogos.

---

### 3. Otimizações de SDXL, IA Local & DSL / DirectStorage (`-sdxl`, `[13]`)
* **Variáveis CUDA / PyTorch Low-VRAM Automáticas:** Configura no perfil do usuário `CUDA_MODULE_LOADING=LAZY`, `SAFETENSORS_FAST_GPU=1` e (em GPUs `<= 4 GB` VRAM) `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True,garbage_collection_threshold:0.6,max_split_size_mb:128` para evitar falta de memória (`CUDA OOM`) e acelerar o carregamento de *checkpoints* SDXL.
* **Flags Específicas por Arquitetura + Launcher (`Start_SDXL_Optimized.bat`):** Localiza instalações de **ComfyUI, Stable Diffusion WebUI (A1111), Forge, Fooocus e SD.Next** nos discos e prepara os parâmetros ideais (ex.: `--lowvram --opt-sdp-attention --no-half-vae --disable-nan-check` em arquiteturas Pascal ou anteriores, evitando o erro `NaN`/tela preta no VAE de SDXL sem duplicar o peso do modelo com `--no-half`).
* **Monitor Inteligente para Workloads de IA & Windows Game Mode:** Detecta execução pesada de IA/SDXL (`python > 600 MB`, `comfyui`, `ollama`, etc.) elevando energia + GPU para **Boost** durante a geração e retornando automaticamente para **Economia** após o término. Também ativa *Windows Game Mode* (`AllowAutoGameMode` / `AutoGameModeEnabled`) e ajustes de energia/PCIe compatíveis com cargas rápidas de textura e **DirectStorage (DSL/DLSS bridge)**.

---

### 4. Automação de Áudio (Realtek `48 kHz`), Vídeo (OBS Studio), Contraste & Programas do PC
* **Realtek Audio & Áudio do Windows (sem *resampling* pesado):**
  * Reinicia temporariamente o `Audiosrv` durante a calibração completa para atualizar com segurança **todas as chaves `WAVEFORMATEXTENSIBLE` (`0xFFFE`)** ativas acima de `96 kHz` (incluindo `Properties` e `FxProperties`) para o padrão de estúdio **`48000 Hz / 24-bit`**, evitando gargalo de CPU e dessincronização entre jogo, Discord e OBS.
  * Desativa suspensão D3 do chip Realtek (`IdlePowerState`, `ConservationIdleTime`, `PerformanceIdleTime = 0`), garante **Estéreo Real** (`AccessibilityMonoMixState = 0`), bloqueia redução de volume em chamadas (`UserDuckingPreference = 3`) e eleva prioridade MMCSS/`audiodg.exe`.
* **OBS Studio (Calibração por Hardware + 2 Perfis Prontos):**
  * Seleciona automaticamente o melhor encoder por hardware (`obs_nvenc_h264_tex` NVIDIA, `h264_texture_amf` AMD, `obs_qsv11` Intel QSV ou `obs_x264`).
  * Ajusta resolução mod-4 com filtro **`Lanczos`**, **48 kHz Estéreo**, faixas AAC **`192 kbps` / `320 kbps`**, `ProcessPriority = High`, `DisableAudioDucking = true` e desativa `BrowserHWAccel` em GPUs com `<= 4 GB` VRAM ou notebooks.
  * Cria dois perfis dedicados sem apagar suas cenas: **`Antigravity_Jogos_Desempenho`** (`p5`/`hq`/`CQP 20`) e **`Antigravity_Streaming_Qualidade`** (`p5`, `CQP 18`, `CBR 4500–7500 kbps`).
* **Wallpaper Engine, Discord, Steam, Navegadores e Apps Pesados:**
  * Configura **Wallpaper Engine** (`playbackmaximized = "pause"`, `pausevram = true`, 30 fps em VRAM limitada), **Steam** (`GPUAccelWebViews = 0` em VRAM limitada/notebook) e **Discord** (`settings.json` protegendo VRAM).
  * Durante jogos, IA ou streaming, reduz prioridade de navegadores e auxiliares (`Brave, Chrome, Edge, Firefox, Opera, steamwebhelper, SearchIndexer`) para `Idle` e eleva a prioridade de jogos, upscalers, OBS, IA e editores/DAWs (`High` / `AboveNormal`).
* **Predefinições de Áudio/Imagem (`Jogos` vs `Streaming`) e Proteção de Contraste:**
  * Garante **RGB Completo (`0-255`)**, Gama linear `1.0`, remoção de `OverlayTestMode` e `GameDVR_FSEBehaviorMode = 0` com proteção **DDC/CI** para evitar imagem lavada / filtro branco.
* **Reparo Automático de Erros do Windows (`-reparar` / `[14]`):**
  * Força `PCIe ASPM = Desligado` e `Disk Idle = 0` em todos os planos de Desktop (evita quedas de barramento GPU `BusReset TDR` e erros de E/S em disco).
  * Remove serviços de drivers órfãos com `.sys` ausente (`SCM 7000`), limpa e recompila `Autorecover MOFs` + repositório WMI, corrige conflito VBS/HVCI (`Kernel-Boot 124`), instala drivers oficiais `PCI\VEN_` pendentes via Windows Update, agenda reparo offline seguro de NTFS no próximo boot (`fsutil dirty set`) se houver corrupção recente e executa `DISM` + `sfc /scannow` sem usar varreduras online que possam congelar o sistema.
* **Monitor Automático em Segundo Plano (`Auto` ou Nível Fixo):**
  * **Leve** para aplicativos pesados desconhecidos (`> 550 MB` RAM).
  * **Médio** para jogos comuns, **OBS Studio** ou editores/DAWs (**Premiere, DaVinci Resolve, Vegas, After Effects, CapCut, Blender, FL Studio, Audacity, Reaper, Photoshop, Visual Studio**, etc.).
  * **Boost** para jogos AAA ultra-pesados, renderização/IA (**SDXL/ComfyUI/Python IA**) ou jogo + OBS simultâneos.
  * Retorna sozinho para **Economia (`Slot 5`)** 12 segundos após fechar os programas pesados.

---

## Como usar

### Pelo Menu Interativo (clique duplo)
Execute **`Modo_Jogo_Automatico.bat`** como Administrador e escolha:
* **`[1]` Calibrar Tudo** *(4 Perfis UV+OC + OBS + Realtek + FSR/SDXL + Fix ASPM)* e ativar **Desempenho Boost** *(padrão principal)*
* **`[2]`** Ativar **Desempenho Médio** (`Slot 3`) — escolha Predefinição Jogos ou Streaming
* **`[3]`** Ativar **Desempenho Boost** (`Slot 4`) — escolha Predefinição Jogos ou Streaming
* **`[4]`** Ativar **Modo Economia** (`Slot 5` + plano econômico)
* **`[5]`** Alternar **Predefinições Áudio/Imagem** (*Jogos* vs *Streaming*) e recalibrar programas
* **`[6]`** Configurar **Nível Automático** (*Auto Inteligente* ou Fixo em *Leve / Médio / Boost*)
* **`[7]`** Iniciar Monitor Automático nesta janela *(jogos, OBS, IA/SDXL e apps)*
* **`[8]`** Instalar Automação no Windows (*inicia oculto no logon via Tarefa Agendada `ModoJogoAutomatico_MSIAfterburner`*)
* **`[9]`** Desinstalar Automação e restaurar backups (`.bak_modojogo` do Afterburner, OBS, Discord, Wallpaper Engine, CS2 e Overwatch)
* **`[10]`** Corrigir Contraste / Filtro Branco agora (`RGB Full 0-255`, Gama `1.0` e DDC/CI)
* **`[11]`** Exibir Diagnóstico Completo (Peças, 4 Perfis GPU, Áudio, OBS, FSR e SDXL)
* **`[12]`** **AMD FSR Universal + Jogos** (calibrar CS2/Overwatch + instalar Magpie & OptiScaler)
* **`[13]`** **Otimizações SDXL & IA** (variáveis CUDA Lazy, PyTorch Low-VRAM e launchers WebUI/ComfyUI)
* **`[14]`** **Reparar Erros do Windows** (ASPM, Drivers Órfãos, WMI, HVCI 124, Drivers PCI WU, Boot NTFS seguro & DISM/SFC)

### Parâmetros diretos via CMD / Atalho
```cmd
Modo_Jogo_Automatico.bat -calibrar
Modo_Jogo_Automatico.bat -leve
Modo_Jogo_Automatico.bat -medio
Modo_Jogo_Automatico.bat -boost
Modo_Jogo_Automatico.bat -desempenho
Modo_Jogo_Automatico.bat -economia
Modo_Jogo_Automatico.bat -presetjogos
Modo_Jogo_Automatico.bat -presetstream
Modo_Jogo_Automatico.bat -fsr
Modo_Jogo_Automatico.bat -instalarfsr
Modo_Jogo_Automatico.bat -sdxl
Modo_Jogo_Automatico.bat -reparar
Modo_Jogo_Automatico.bat -monitor
Modo_Jogo_Automatico.bat -auto
Modo_Jogo_Automatico.bat -corrigircor
```

---

## Código Completo (`Modo_Jogo_Automatico.bat`)

```bat@echo off
chcp 65001 >nul
title Modo Jogo Automatico - Hardware-Aware + Perfis MSI Afterburner

set "MODOJOGO_ARGS=%~1"
set "MODOJOGO_SELF=%~f0"

:: Se nao foi chamado em modo monitor silencioso, verifica elevacao de Administrador
if /i "%MODOJOGO_ARGS%"=="-auto" goto :RunPS
if /i "%MODOJOGO_ARGS%"=="-monitor" goto :RunPS

net session >nul 2>&1
if %errorLevel% neq 0 (
    echo [!] Solicitando permissao de Administrador para configurar perfis, energia e automacao...
    if "%MODOJOGO_ARGS%"=="" (
        powershell -NoProfile -ExecutionPolicy Bypass -Command "Start-Process -FilePath $env:MODOJOGO_SELF -Verb RunAs"
    ) else (
        powershell -NoProfile -ExecutionPolicy Bypass -Command "Start-Process -FilePath $env:MODOJOGO_SELF -ArgumentList $env:MODOJOGO_ARGS -Verb RunAs"
    )
    exit /b
)

:RunPS
powershell -NoProfile -ExecutionPolicy Bypass -Command "$lines = [System.IO.File]::ReadAllLines($env:MODOJOGO_SELF, [System.Text.Encoding]::UTF8); $idx = [Array]::IndexOf($lines, ':: === INICIO_POWERSHELL ==='); $code = $lines[($idx+1)..($lines.Length-1)] -join [Environment]::NewLine; & ([ScriptBlock]::Create($code))"
exit /b

:: === INICIO_POWERSHELL ===
$ErrorActionPreference = 'SilentlyContinue'

# ============================================================================
# CONFIGURACOES OPCIONAIS DO USUARIO (0 = Automatico e 100% seguro via hardware)
# ============================================================================
$SlotBackupOriginal = 1      # Slot preservado intacto (referencia do usuario)
$SlotLeve           = 2      # Slot MSI Afterburner: DESEMPENHO LEVE (Undervolt + Leve OC)
$SlotMedio          = 3      # Slot MSI Afterburner: DESEMPENHO MEDIO (Undervolt + Medio OC)
$SlotBoost          = 4      # Slot MSI Afterburner: DESEMPENHO BOOST (Undervolt + Boost OC Seguro)
$EcoSlot            = 5      # Slot MSI Afterburner: ECONOMIA (Undervolt Eficiencia)
$DefaultAutoTier    = 'Auto' # 'Auto' (inteligente por programa), 'Leve', 'Medio' ou 'Boost'
$PerfCoreOffsetMHz  = 0      # 0 = automatico por hardware (tier Leve); >0 sobrescreve base Core
$PerfMemOffsetMHz   = 0      # 0 = automatico por hardware (tier Leve); >0 sobrescreve base VRAM
$AutoFixContrast    = $true  # $true corrige automaticamente filtro branco/contraste (RGB Full, Gama, DDC)
$AutoOptimizeApps   = $true  # $true otimiza OBS, Realtek/Audio, Discord, Wallpaper Engine e apps pesados
$PollIntervalSec    = 4      # Intervalo de verificacao do monitor automatico (segundos)
$EcoGraceSec        = 12     # Tempo apos fechar jogos/programas pesados antes de ir para Economia
$CloseBrowsersAuto  = $false # $true fecha navegadores em modo automatico; $false apenas reduz prioridade
# ============================================================================

$TaskName    = "ModoJogoAutomatico_MSIAfterburner"
$InstallDir  = Join-Path $env:ProgramData "ModoJogoAutomatico"
$ConfigFile  = Join-Path $InstallDir "config.json"
$LogFile     = Join-Path $env:TEMP "ModoJogoAutomatico.log"

$KnownGameExes = @(
    'cs2','csgo','valorant','valorant-win64-shipping','fortniteclient-win64-shipping',
    'overwatch','r5apex','gta5','playgtav','gtaiv','cyberpunk2077','witcher3',
    'eldenring','rdr2','bf4','bf1','bfv','bf2042','forzahorizon4','forzahorizon5',
    'tslgame','dota2','league of legends','rainbowsix','escapefromtarkov',
    'helldivers2','once_human','acs','metro','eurotrucks2','farcry3','fc3_d3d11',
    'cod','bg3','bg3_dx11','starfield','hogwartslegacy','palworld','rustclient',
    'dayz_x64','hl2','rocketleague','marvelrivals','deadlock','apex','destiny2',
    'pubg','warframe.x64','re4','re5dx9','isaac-ng','repo','hades','oriwotw',
    'daysgone','assettocorsa','drift','readyornot-win64-shipping'
)

$UltraHeavyGameExes = @(
    'cyberpunk2077','rdr2','eldenring','starfield','hogwartslegacy','bf2042',
    'helldivers2','palworld','daysgone','readyornot-win64-shipping','forzahorizon5',
    'flightsimulator','alanwake2','witcher3','gta5'
)

$StreamingExes = @(
    'obs64','obs32','streamlabs obs','xsplit.core','twitchstudio','vmix64','nvidia broadcast'
)

$CreationWorkExes = @(
    'adobe premiere pro','premiere pro','resolve','vegas210','vegas200','vegas190','vegas180',
    'afterfx','blender','capcut','audacity','fl64','fl','reaper','handbrake','shotcut',
    'kdenlive','photoshop','gimp-2.10','devenv','rider64'
)

$AiWorkloadExes = @(
    'comfyui','ollama','ollama app','llama-server','koboldcpp','python','pythonw'
)

$UpscalerExes = @(
    'losslessscaling','magpie'
)

$AudioConsoleExes = @(
    'realtekaudiocontrol','rtkngui64','ravcpl64','voicemeeter','voicemeeterpro','voicemeeter8',
    'steelseriesgg','nvidia broadcast'
)

$BackgroundThrottleBrowsers = @('brave','chrome','msedge','firefox','opera')
$BackgroundThrottleHelpers  = @('steamwebhelper','SearchIndexer','DSATray','epicwebhelper')

$IgnoredExeNames = @(
    'unins000','uninstall','setup','installer','vc_redist','dxsetup','crashhandler',
    'crashreporter','updater','launcher','bootstrap','anticheat','easyanticheat_eos_setup',
    'battleye','vgc','vgtray','steam','steamwebhelper','steamservice','epicwebhelper',
    'brave','chrome','msedge','firefox','opera','discord','obs64','obs32','rtss','msiafterburner',
    'explorer','dwm','csrss','svchost','lsass','wininit','services','taskmgr','conhost',
    'cmd','powershell','pwsh','applicationframehost','shellexperiencehost','searchhost',
    'startmenuexperiencehost','textinputhost','runtimebroker','audiodg','rtkauduservice64',
    'nvidia overlay','nvcontainer','code','antigravity','windowsterminal','losslessscaling','magpie'
)

function Get-ModoJogoConfig {
    $cfg = [PSCustomObject]@{
        AutoPerfTier   = $DefaultAutoTier
        ActiveAVPreset = 'Jogos'
    }
    if (Test-Path $ConfigFile) {
        try {
            $raw = Get-Content -Path $ConfigFile -Raw -Encoding UTF8 | ConvertFrom-Json
            if ($raw.AutoPerfTier -eq 'Alto') { $cfg.AutoPerfTier = 'Boost' }
            elseif ($raw.AutoPerfTier -in @('Auto','Leve','Medio','Boost')) { $cfg.AutoPerfTier = $raw.AutoPerfTier }
            if ($raw.ActiveAVPreset -in @('Jogos','Streaming','Economia')) { $cfg.ActiveAVPreset = $raw.ActiveAVPreset }
        } catch {}
    }
    return $cfg
}

function Save-ModoJogoConfig {
    param([string]$AutoPerfTier, [string]$ActiveAVPreset)
    $cur = Get-ModoJogoConfig
    if ($AutoPerfTier -eq 'Alto') { $AutoPerfTier = 'Boost' }
    if ($AutoPerfTier -in @('Auto','Leve','Medio','Boost')) { $cur.AutoPerfTier = $AutoPerfTier }
    if ($ActiveAVPreset -in @('Jogos','Streaming','Economia')) { $cur.ActiveAVPreset = $ActiveAVPreset }
    try {
        if (-not (Test-Path $InstallDir)) { New-Item -Path $InstallDir -ItemType Directory -Force | Out-Null }
        ($cur | ConvertTo-Json) | Set-Content -Path $ConfigFile -Encoding UTF8
    } catch {}
    return $cur
}

function Write-Log {
    param([string]$Msg, [string]$Color = "Cyan", [switch]$Silent)
    $line = "[{0}] {1}" -f (Get-Date -Format "HH:mm:ss"), $Msg
    if (-not $Silent) { Write-Host $line -ForegroundColor $Color }
    Add-Content -Path $LogFile -Value $line -Encoding UTF8
    $fi = Get-Item $LogFile -ErrorAction SilentlyContinue
    if ($fi -and $fi.Length -gt 512KB) {
        $tail = Get-Content $LogFile -Tail 200
        Set-Content -Path $LogFile -Value $tail -Encoding UTF8
    }
}

function Test-IsAdmin {
    $id = [Security.Principal.WindowsIdentity]::GetCurrent()
    $p  = New-Object Security.Principal.WindowsPrincipal($id)
    return $p.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
}

function Get-NvidiaSmiPath {
    $cmd = Get-Command nvidia-smi -ErrorAction SilentlyContinue
    if ($cmd) { return $cmd.Source }
    $fallback = Join-Path $env:SystemRoot "System32\nvidia-smi.exe"
    if (Test-Path $fallback) { return $fallback }
    return $null
}

function Get-HardwareProfile {
    $cpuObj  = Get-CimInstance Win32_Processor | Select-Object -First 1
    $osObj   = Get-CimInstance Win32_OperatingSystem
    $gpus    = @(Get-CimInstance Win32_VideoController | Where-Object { $_.Name -notmatch 'Microsoft Basic|Parsec|Indirect|Virtual' })
    $gpuMain = $gpus | Sort-Object AdapterRAM -Descending | Select-Object -First 1
    $isLaptop = ($null -ne (Get-CimInstance Win32_Battery))

    $ramTotalGB = [math]::Round($osObj.TotalVisibleMemorySize / 1MB, 1)
    $ramFreeGB  = [math]::Round($osObj.FreePhysicalMemory / 1MB, 1)

    $vramMB = 0
    if ($gpuMain -and $gpuMain.AdapterRAM) {
        $vramMB = [math]::Round($gpuMain.AdapterRAM / 1MB, 0)
    }

    $vendor = "Other"
    if ($gpuMain.Name -match 'NVIDIA') { $vendor = "NVIDIA" }
    elseif ($gpuMain.Name -match 'AMD|Radeon') { $vendor = "AMD" }
    elseif ($gpuMain.Name -match 'Intel') { $vendor = "Intel" }

    $nvSmi = Get-NvidiaSmiPath
    $nvMinPct = if ($isLaptop) { 80 } else { 70 }
    $nvMaxPct = 100
    $nvTempC  = $null

    if ($vendor -eq "NVIDIA" -and $nvSmi) {
        $q = & $nvSmi --query-gpu=memory.total,power.min_limit,power.default_limit,power.max_limit,temperature.gpu --format=csv,noheader,nounits 2>$null | Select-Object -First 1
        if ($q) {
            $parts = $q -split ',\s*'
            if ($parts.Count -ge 5) {
                $inv = [System.Globalization.CultureInfo]::InvariantCulture
                $numStyle = [System.Globalization.NumberStyles]::Float
                $mTot = 0.0; if ([double]::TryParse($parts[0].Trim(), $numStyle, $inv, [ref]$mTot) -and $mTot -gt 0) { $vramMB = [int][math]::Round($mTot) }
                $pMin = 0.0; $pDef = 0.0; $pMax = 0.0
                [void][double]::TryParse($parts[1].Trim(), $numStyle, $inv, [ref]$pMin)
                [void][double]::TryParse($parts[2].Trim(), $numStyle, $inv, [ref]$pDef)
                [void][double]::TryParse($parts[3].Trim(), $numStyle, $inv, [ref]$pMax)
                if ($pDef -gt 0 -and $pMin -gt 0) {
                    $calcMin = [int][math]::Round(($pMin / $pDef) * 100)
                    $nvMinPct = [math]::Max(65, [math]::Min(85, $calcMin))
                }
                if ($pDef -gt 0 -and $pMax -gt 0 -and (-not $isLaptop)) {
                    $calcMax = [int][math]::Round(($pMax / $pDef) * 100)
                    $nvMaxPct = [math]::Max(100, [math]::Min(108, $calcMax))
                }
                $tTmp = 0; if ([int]::TryParse($parts[4].Trim(), [ref]$tTmp)) { $nvTempC = $tTmp }
            }
        }
    }

    $disks = @(Get-PhysicalDisk | Select-Object FriendlyName, MediaType, @{N='GB';E={[math]::Round($_.Size/1GB,0)}})
    $isPascalOrOlder = ($vendor -eq 'NVIDIA') -and ($gpuMain.Name -match 'GTX\s*(7\d\d|8\d\d|9\d\d|10\d\d|16\d\d)|GT\s*\d+|MX\d+|TITAN\s*(X|Xp|V)?\b')
    $noTensorCores   = ($vendor -ne 'NVIDIA') -or $isPascalOrOlder
    $dispW = if ($gpuMain.CurrentHorizontalResolution -gt 0) { [int]$gpuMain.CurrentHorizontalResolution } else { 1920 }
    $dispH = if ($gpuMain.CurrentVerticalResolution -gt 0) { [int]$gpuMain.CurrentVerticalResolution } else { 1080 }
    $dispHz = if ($gpuMain.CurrentRefreshRate -gt 0) { [int]$gpuMain.CurrentRefreshRate } else { 60 }

    return [PSCustomObject]@{
        CpuName         = ($cpuObj.Name -replace '\s+', ' ').Trim()
        CpuCores        = $cpuObj.NumberOfCores
        CpuThreads      = $cpuObj.NumberOfLogicalProcessors
        RamTotalGB      = $ramTotalGB
        RamFreeGB       = $ramFreeGB
        IsLaptop        = $isLaptop
        GpuName         = $gpuMain.Name
        GpuVendor       = $vendor
        VramMB          = $vramMB
        LowVram         = ($vramMB -gt 0 -and $vramMB -le 4200)
        IsPascalOrOlder = $isPascalOrOlder
        NoTensorCores   = $noTensorCores
        DisplayW        = $dispW
        DisplayH        = $dispH
        DisplayHz       = $dispHz
        PerfPowerPct    = $nvMaxPct
        EcoPowerPct     = $nvMinPct
        GpuTempC        = $nvTempC
        Disks           = $disks
    }
}

function Find-Afterburner {
    $candidates = @(
        (Join-Path ${env:ProgramFiles(x86)} "MSI Afterburner\MSIAfterburner.exe"),
        (Join-Path $env:ProgramFiles "MSI Afterburner\MSIAfterburner.exe")
    )
    foreach ($c in $candidates) {
        if ($c -and (Test-Path $c)) {
            return [PSCustomObject]@{
                ExePath     = $c
                Dir         = (Split-Path $c -Parent)
                ProfilesDir = (Join-Path (Split-Path $c -Parent) "Profiles")
            }
        }
    }
    return $null
}

function Get-PowerSchemeIds {
    $raw = powercfg /list 2>$null | Out-String
    $ultimate = $null; $high = $null; $balanced = $null; $saver = $null
    foreach ($line in ($raw -split "`r?`n")) {
        if ($line -match '([0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12})') {
            $g = $matches[1]
            if ($line -match 'Desempenho M|Ultimate Performance' -or $g -eq 'e9a42b02-d5df-448d-aa00-03f14749eb61') { $ultimate = $g }
            elseif ($line -match 'Alto desempenho|High performance' -or $g -eq '8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c') { $high = $g }
            elseif ($line -match 'Equilibrado|Balanced' -or $g -eq '381b4222-f694-41f0-9685-ff5bb260df2e') { $balanced = $g }
            elseif ($line -match 'Economia de energia|Power saver' -or $g -eq 'a1841308-3541-4fab-bc81-f71556f20b4a') { $saver = $g }
        }
    }
    if (-not $ultimate -and -not $high) {
        $dup = powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61 2>$null | Out-String
        if ($dup -match '([0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12})') { $ultimate = $matches[1] }
    }
    return [PSCustomObject]@{
        PerfGuid = if ($ultimate) { $ultimate } elseif ($high) { $high } else { $balanced }
        EcoGuid  = if ($saver) { $saver } else { $balanced }
    }
}

function Ensure-SafeDesktopPowerSettings {
    param($Hw)
    if (-not $Hw) { $Hw = Get-HardwareProfile }
    if ($Hw.IsLaptop -or (-not (Test-IsAdmin))) { return }
    try {
        foreach ($sl in (powercfg /list)) {
            if ($sl -match '([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})') {
                $g = $matches[1]
                # Desliga PCIe ASPM (evita nvlddmkm 153 BusReset TDR em GPUs PCIe 75W) e evita spin-down agressivo de HDD (disk 153)
                powercfg /setacvalueindex $g 501a4d13-42af-4429-9fd1-a8218c268e20 ee12f906-d277-404b-b6da-e5fa1a576df5 0 2>$null | Out-Null
                powercfg /setdcvalueindex $g 501a4d13-42af-4429-9fd1-a8218c268e20 ee12f906-d277-404b-b6da-e5fa1a576df5 0 2>$null | Out-Null
                powercfg /setacvalueindex $g 0012ee47-9041-4b5d-9b77-535fba8b1442 6738e2c4-e8a5-4a42-b16a-e040e769756e 0 2>$null | Out-Null
            }
        }
    } catch {}
}

function Set-IniSection {
    param(
        [string]$FilePath,
        [string]$SectionName,
        [string[]]$KeyValueLines
    )
    $enc = [System.Text.Encoding]::Default
    $lines = [System.Collections.Generic.List[string]]::new([System.IO.File]::ReadAllLines($FilePath, $enc))
    $header = "[$SectionName]"
    $startIdx = -1
    $endIdx   = $lines.Count

    for ($i = 0; $i -lt $lines.Count; $i++) {
        if ($lines[$i].Trim() -ieq $header) {
            $startIdx = $i
            for ($j = $i + 1; $j -lt $lines.Count; $j++) {
                if ($lines[$j].Trim() -match '^\[.+\]$') {
                    $endIdx = $j
                    break
                }
            }
            break
        }
    }

    if ($startIdx -ge 0) {
        $lines.RemoveRange($startIdx, $endIdx - $startIdx)
        $insertAt = $startIdx
    } else {
        $insertAt = $lines.Count
        for ($i = 0; $i -lt $lines.Count; $i++) {
            if ($lines[$i].Trim() -match '^\[(Defaults|Settings)\]$') {
                $insertAt = $i
                break
            }
        }
    }

    $block = @($header) + $KeyValueLines
    $lines.InsertRange($insertAt, [string[]]$block)
    [System.IO.File]::WriteAllLines($FilePath, $lines.ToArray(), $enc)
}

function Get-IniSectionValue {
    param([string]$FilePath, [string]$SectionName, [string]$KeyName)
    if (-not (Test-Path $FilePath)) { return $null }
    $header = "[$SectionName]"
    $inSec = $false
    foreach ($line in [System.IO.File]::ReadAllLines($FilePath, [System.Text.Encoding]::Default)) {
        $t = $line.Trim()
        if ($t -match '^\[.+\]$') {
            if ($inSec) { break }
            $inSec = ($t -ieq $header)
            continue
        }
        if ($inSec -and $t -match ("^" + [regex]::Escape($KeyName) + "\s*=\s*(.+)$")) {
            return $matches[1].Trim()
        }
    }
    return $null
}

function Get-GpuTuningPlan {
    param($Hw)
    $isDedicated = ($Hw.VramMB -ge 1500 -and $Hw.GpuVendor -in @('NVIDIA','AMD'))
    $baseCore = 0; $baseMem = 0; $ecoBoost = 0
    $medCoreMult = 1.50; $medMemMult = 1.67
    $highCoreMult = 1.88; $highMemMult = 2.17

    if ($isDedicated) {
        if ($Hw.IsLaptop) {
            $baseCore = 25; $baseMem = 100; $ecoBoost = 15
            $medCoreMult = 1.40; $medMemMult = 1.50
            $highCoreMult = 1.80; $highMemMult = 2.00
        } elseif ($Hw.GpuVendor -eq 'NVIDIA') {
            if ($Hw.VramMB -ge 6000) {
                $baseCore = 50; $baseMem = 200; $ecoBoost = 25
            } elseif ($Hw.VramMB -ge 3500) {
                $baseCore = 40; $baseMem = 150; $ecoBoost = 25
            } else {
                $baseCore = 30; $baseMem = 125; $ecoBoost = 20
            }
        } else {
            $baseCore = 35; $baseMem = 150; $ecoBoost = 15
            $medCoreMult = 1.40; $medMemMult = 1.50
            $highCoreMult = 1.80; $highMemMult = 2.00
        }
    }

    if ($PerfCoreOffsetMHz -gt 0) { $baseCore = $PerfCoreOffsetMHz }
    if ($PerfMemOffsetMHz  -gt 0) { $baseMem  = $PerfMemOffsetMHz }

    $round5 = { param([double]$val) [int]([math]::Round($val / 5.0) * 5) }
    $leveCore  = $baseCore; $leveMem = $baseMem
    $medioCore = (& $round5 ($baseCore * $medCoreMult))
    $medioMem  = (& $round5 ($baseMem  * $medMemMult))
    $boostCore = (& $round5 ($baseCore * $highCoreMult))
    $boostMem  = (& $round5 ($baseMem  * $highMemMult))

    $safeBoostPL = if ($Hw.IsLaptop) { 100 } else { $Hw.PerfPowerPct }
    $safeMedPL   = if ($Hw.IsLaptop) { 100 } else { [math]::Min($Hw.PerfPowerPct, 104) }
    $thermBase   = if ($Hw.IsLaptop) { 80 } else { 83 }

    return [PSCustomObject]@{
        IsDedicated = $isDedicated
        EcoBoostMHz = $ecoBoost
        EcoVoltFrac = if ($Hw.IsLaptop) { 0.56 } else { 0.58 }
        Tiers       = @{
            Leve = [PSCustomObject]@{
                Slot = $SlotLeve; Name = 'Leve'; CoreMHz = $leveCore; MemMHz = $leveMem
                PowerLimit = 100; ThermalLimit = $thermBase; FanMode = 0; FanSpeed = 52
                VoltFrac = if ($Hw.IsLaptop) { 0.78 } else { 0.82 }; PeakExtra = if ($Hw.IsLaptop) { 0 } else { 0 }
            }
            Medio = [PSCustomObject]@{
                Slot = $SlotMedio; Name = 'Medio'; CoreMHz = $medioCore; MemMHz = $medioMem
                PowerLimit = $safeMedPL; ThermalLimit = $thermBase; FanMode = 0; FanSpeed = 62
                VoltFrac = if ($Hw.IsLaptop) { 0.85 } else { 0.89 }; PeakExtra = if ($Hw.IsLaptop) { 10 } else { 20 }
            }
            Boost = [PSCustomObject]@{
                Slot = $SlotBoost; Name = 'Boost'; CoreMHz = $boostCore; MemMHz = $boostMem
                PowerLimit = $safeBoostPL; ThermalLimit = $thermBase; FanMode = 0; FanSpeed = 72
                VoltFrac = if ($Hw.IsLaptop) { 0.90 } else { 0.92 }; PeakExtra = if ($Hw.IsLaptop) { 20 } else { 35 }
            }
        }
    }
}

function New-UndervoltOverclockVFCurve {
    param(
        [string]$DefaultsHex,
        [double]$CoreBoostMHz,
        [double]$VoltCapFrac,
        [double]$PeakExtraMHz = 0,
        [bool]$MatchPeakClk = $false
    )
    if ([string]::IsNullOrWhiteSpace($DefaultsHex) -or $DefaultsHex.Length -lt 48 -or ($DefaultsHex.Length % 2) -ne 0) {
        return $null
    }

    $bytes = [byte[]]::new($DefaultsHex.Length / 2)
    for ($i = 0; $i -lt $bytes.Length; $i++) {
        $bytes[$i] = [Convert]::ToByte($DefaultsHex.Substring($i * 2, 2), 16)
    }

    $cnt = [BitConverter]::ToUInt32($bytes, 4)
    if ($cnt -lt 20 -or (12 + $cnt * 12) -gt $bytes.Length) { return $null }

    $v = [double[]]::new($cnt)
    $c = [double[]]::new($cnt)
    $maxC = 0.0
    for ($i = 0; $i -lt $cnt; $i++) {
        $off = 12 + $i * 12
        $v[$i] = [double][BitConverter]::ToSingle($bytes, $off)
        $c[$i] = [double][BitConverter]::ToSingle($bytes, $off + 4)
        if ($c[$i] -gt $maxC) { $maxC = $c[$i] }
    }

    $peakIdx = $cnt - 1
    for ($i = 0; $i -lt $cnt; $i++) {
        if ($c[$i] -ge ($maxC - 0.5)) { $peakIdx = $i; break }
    }

    $minV = $v[0]; $peakV = $v[$peakIdx]
    $targetV = $minV + $VoltCapFrac * ($peakV - $minV)

    $capIdx = 0; $bestDiff = 1e9
    for ($i = 0; $i -lt $cnt; $i++) {
        $d = [math]::Abs($v[$i] - $targetV)
        if ($d -lt $bestDiff) { $bestDiff = $d; $capIdx = $i }
    }

    if ($MatchPeakClk) {
        $goalClk = $maxC + $PeakExtraMHz
        $foundIdx = -1
        for ($i = 0; $i -lt $peakIdx; $i++) {
            if (($c[$i] + $CoreBoostMHz) -ge $goalClk) { $foundIdx = $i; break }
        }
        $minIdx = [math]::Max(10, [int][math]::Floor($peakIdx * 0.76))
        $maxIdx = [math]::Max($minIdx, $peakIdx - 2)
        $desiredIdx = if ($foundIdx -ge 0) { [math]::Max($capIdx, $foundIdx) } else { $capIdx }
        $capIdx = [math]::Max($minIdx, [math]::Min($maxIdx, $desiredIdx))
    } else {
        $capIdx = [math]::Max(8, [math]::Min($peakIdx - 5, $capIdx))
    }

    $plateauClk = $c[$capIdx] + $CoreBoostMHz
    $out = [byte[]]::new($bytes.Length)
    [Array]::Copy($bytes, $out, $bytes.Length)
    [BitConverter]::GetBytes([single]0.0).CopyTo($out, 8)

    $rampStartV = $minV + 60.0
    $rampFullV  = $minV + 180.0
    for ($i = 0; $i -lt $cnt; $i++) {
        $off = 12 + $i * 12
        if ($i -le $capIdx) {
            if ($v[$i] -le $rampStartV) { $ptOff = 0.0 }
            elseif ($v[$i] -ge $rampFullV) { $ptOff = $CoreBoostMHz }
            else { $ptOff = $CoreBoostMHz * (($v[$i] - $rampStartV) / ($rampFullV - $rampStartV)) }
        } else {
            $ptOff = $plateauClk - $c[$i]
        }
        $ptOff = [math]::Round($ptOff * 2.0) / 2.0
        [BitConverter]::GetBytes([single]$ptOff).CopyTo($out, $off + 8)
    }

    $sb = [System.Text.StringBuilder]::new($out.Length * 2)
    foreach ($b in $out) { [void]$sb.Append($b.ToString("X2")) }

    return [PSCustomObject]@{
        Hex         = $sb.ToString()
        CapMv       = [int][math]::Round($v[$capIdx])
        PlateauMHz  = [int][math]::Round($plateauClk)
        MaxStockMv  = [int][math]::Round($v[$cnt - 1])
        MaxStockMHz = [int][math]::Round($maxC)
    }
}

function Ensure-DisplayContrastAndColor {
    param([switch]$SkipGammaRamp, [switch]$Silent)
    if (-not $AutoFixContrast) { return }

    try {
        Remove-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\Dwm' -Name 'OverlayTestMode' -ErrorAction SilentlyContinue
        $gcPath = 'HKCU:\System\GameConfigStore'
        if (Test-Path $gcPath) {
            Set-ItemProperty -Path $gcPath -Name 'GameDVR_FSEBehaviorMode' -Value 0 -Type DWord -ErrorAction SilentlyContinue
            Remove-ItemProperty -Path $gcPath -Name 'GameDVR_HonorUserFSEBehaviorMode' -ErrorAction SilentlyContinue
        }
        $cfPath = 'HKCU:\Software\Microsoft\ColorFiltering'
        if (Test-Path $cfPath) {
            Set-ItemProperty -Path $cfPath -Name 'Active' -Value 0 -Type DWord -ErrorAction SilentlyContinue
        }
    } catch {}

    if (-not ([System.Management.Automation.PSTypeName]'ModoJogoDisplayHelper').Type) {
        $csDisp = @'
using System;
using System.Runtime.InteropServices;

public static class ModoJogoDisplayHelper {
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
    public struct PHYSICAL_MONITOR {
        public IntPtr hPhysicalMonitor;
        [MarshalAs(LayoutKind.ByValTStr, SizeConst = 128)]
        public string szPhysicalMonitorDescription;
    }

    public delegate bool MonitorEnumProc(IntPtr hMonitor, IntPtr hdcMonitor, IntPtr lprcMonitor, IntPtr dwData);

    [DllImport("user32.dll")] static extern bool EnumDisplayMonitors(IntPtr hdc, IntPtr lprcClip, MonitorEnumProc lpfnEnum, IntPtr dwData);
    [DllImport("dxva2.dll")] static extern bool GetNumberOfPhysicalMonitorsFromHMONITOR(IntPtr hMonitor, out uint pdwNumberOfPhysicalMonitors);
    [DllImport("dxva2.dll")] static extern bool GetPhysicalMonitorsFromHMONITOR(IntPtr hMonitor, uint dwPhysicalMonitorArraySize, [Out] PHYSICAL_MONITOR[] pPhysicalMonitorArray);
    [DllImport("dxva2.dll")] static extern bool DestroyPhysicalMonitors(uint dwPhysicalMonitorArraySize, PHYSICAL_MONITOR[] pPhysicalMonitorArray);
    [DllImport("dxva2.dll")] static extern bool GetVCPFeatureAndVCPFeatureReply(IntPtr hMonitor, byte bVCPCode, out uint pvct, out uint pdwCurrentValue, out uint pdwMaximumValue);
    [DllImport("dxva2.dll")] static extern bool SetVCPFeature(IntPtr hMonitor, byte bVCPCode, uint dwNewValue);
    [DllImport("gdi32.dll")] static extern IntPtr CreateDC(string lpszDriver, string lpszDevice, string lpszOutput, IntPtr lpInitData);
    [DllImport("gdi32.dll")] static extern bool DeleteDC(IntPtr hdc);
    [DllImport("gdi32.dll")] static extern bool SetDeviceGammaRamp(IntPtr hdc, ushort[] lpRamp);
    [DllImport("kernel32.dll")] static extern IntPtr LoadLibrary(string lpLibFileName);
    [DllImport("kernel32.dll")] static extern IntPtr GetProcAddress(IntPtr hModule, string lpProcName);

    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] delegate IntPtr NvQueryInterface(uint id);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] delegate int NvInit();
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] delegate int NvEnumGpus(IntPtr[] handles, out int count);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] delegate int NvGetDispIds(IntPtr gpu, [Out] byte[] buf, ref uint count, uint flags);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] delegate int NvDispColorCtrl(uint displayId, byte[] colorData);

    public static void ResetLinearGamma() {
        IntPtr hdc = CreateDC("DISPLAY", null, null, IntPtr.Zero);
        if (hdc == IntPtr.Zero) return;
        try {
            ushort[] ramp = new ushort[768];
            for (int i = 0; i < 256; i++) {
                ushort v = (ushort)(i * 257);
                ramp[i] = v; ramp[i + 256] = v; ramp[i + 512] = v;
            }
            SetDeviceGammaRamp(hdc, ramp);
        } finally { DeleteDC(hdc); }
    }

    public static int EnforceNvidiaFullRangeRgb() {
        int fixedCnt = 0;
        try {
            IntPtr mod = LoadLibrary("nvapi64.dll");
            if (mod == IntPtr.Zero) return 0;
            IntPtr qiPtr = GetProcAddress(mod, "nvapi_QueryInterface");
            if (qiPtr == IntPtr.Zero) return 0;
            var qi = Marshal.GetDelegateForFunctionPointer<NvQueryInterface>(qiPtr);
            IntPtr initPtr = qi(0x0150E828);
            IntPtr enumPtr = qi(0xE5AC921F);
            IntPtr dispPtr = qi(0x0078DBA2);
            IntPtr colPtr  = qi(0x92F9D80D);
            if (initPtr == IntPtr.Zero || enumPtr == IntPtr.Zero || dispPtr == IntPtr.Zero || colPtr == IntPtr.Zero) return 0;
            if (Marshal.GetDelegateForFunctionPointer<NvInit>(initPtr)() != 0) return 0;

            var enumGpus = Marshal.GetDelegateForFunctionPointer<NvEnumGpus>(enumPtr);
            var getDisp  = Marshal.GetDelegateForFunctionPointer<NvGetDispIds>(dispPtr);
            var colCtrl  = Marshal.GetDelegateForFunctionPointer<NvDispColorCtrl>(colPtr);

            IntPtr[] gpus = new IntPtr[64];
            int gpuCount = 0;
            if (enumGpus(gpus, out gpuCount) != 0) return 0;

            for (int g = 0; g < gpuCount; g++) {
                uint cnt = 0;
                if (getDisp(gpus[g], null, ref cnt, 0) != 0 || cnt == 0) continue;
                byte[] buf = new byte[16 * cnt];
                for (uint i = 0; i < cnt; i++) BitConverter.GetBytes((1u << 16) | 16u).CopyTo(buf, (int)(i * 16));
                if (getDisp(gpus[g], buf, ref cnt, 0) != 0) continue;

                for (int i = 0; i < cnt; i++) {
                    uint did = BitConverter.ToUInt32(buf, i * 16 + 8);
                    if (did == 0) continue;
                    byte[] c = new byte[24];
                    BitConverter.GetBytes((5u << 16) | 24u).CopyTo(c, 0);
                    BitConverter.GetBytes((ushort)24).CopyTo(c, 4);
                    c[6] = 1; // GET
                    if (colCtrl(did, c) == 0 && (c[7] != 0 || c[9] != 0)) {
                        c[6] = 2; c[7] = 0; c[8] = 0; c[9] = 0; // RGB + VESA Full (0-255)
                        BitConverter.GetBytes(2u).CopyTo(c, 12); // 8 bpc
                        BitConverter.GetBytes(1u).CopyTo(c, 16); // USER policy
                        if (colCtrl(did, c) == 0) fixedCnt++;
                    }
                }
            }
        } catch {}
        return fixedCnt;
    }

    public static int GuardMonitorDdcContrast() {
        int fixedMonitors = 0;
        try {
            EnumDisplayMonitors(IntPtr.Zero, IntPtr.Zero, delegate(IntPtr hMon, IntPtr hdc, IntPtr rect, IntPtr data) {
                uint num = 0;
                if (!GetNumberOfPhysicalMonitorsFromHMONITOR(hMon, out num) || num == 0) return true;
                var arr = new PHYSICAL_MONITOR[num];
                if (!GetPhysicalMonitorsFromHMONITOR(hMon, num, arr)) return true;
                try {
                    foreach (var pm in arr) {
                        uint t, cCur = 50, cMax = 100, bCur = 80, bMax = 100;
                        bool hasC = GetVCPFeatureAndVCPFeatureReply(pm.hPhysicalMonitor, 0x12, out t, out cCur, out cMax);
                        bool hasB = GetVCPFeatureAndVCPFeatureReply(pm.hPhysicalMonitor, 0x10, out t, out bCur, out bMax);
                        bool changed = false;

                        if (hasC && cMax >= 50 && cCur < 30) {
                            SetVCPFeature(pm.hPhysicalMonitor, 0x12, 50);
                            changed = true;
                        }
                        foreach (byte blCode in new byte[] { 0x6C, 0x6E, 0x70 }) {
                            uint blCur = 50, blMax = 100;
                            if (GetVCPFeatureAndVCPFeatureReply(pm.hPhysicalMonitor, blCode, out t, out blCur, out blMax) && blMax >= 50 && blCur > 55) {
                                SetVCPFeature(pm.hPhysicalMonitor, blCode, 50);
                                changed = true;
                            }
                        }
                        if (changed && hasB && bMax >= 50 && bCur >= 95) {
                            SetVCPFeature(pm.hPhysicalMonitor, 0x10, 80);
                        }
                        if (changed) fixedMonitors++;
                    }
                } finally { DestroyPhysicalMonitors(num, arr); }
                return true;
            }, IntPtr.Zero);
        } catch {}
        return fixedMonitors;
    }
}
'@
        Add-Type -TypeDefinition $csDisp -ErrorAction SilentlyContinue
    }

    try {
        if (-not $SkipGammaRamp) { [ModoJogoDisplayHelper]::ResetLinearGamma() }
        $nvFixed  = [ModoJogoDisplayHelper]::EnforceNvidiaFullRangeRgb()
        $ddcFixed = [ModoJogoDisplayHelper]::GuardMonitorDdcContrast()
        if (($nvFixed -gt 0 -or $ddcFixed -gt 0) -and (-not $Silent)) {
            Write-Log "Contraste/cor restaurados (RGB Full 0-255 ajustes=$nvFixed | DDC monitor ajustes=$ddcFixed)." "Green" -Silent:$Silent
        }
    } catch {}
}

function Ensure-AfterburnerProfiles {
    param($Hw, [switch]$ForceOverwrite)
    $ab = Find-Afterburner
    if (-not $ab -or -not (Test-Path $ab.ProfilesDir)) {
        Write-Log "MSI Afterburner nao encontrado; usando apenas gerenciamento de energia, audio e programas." "Yellow"
        return $false
    }

    $gpuCfgs = @(Get-ChildItem $ab.ProfilesDir -Filter "VEN_*.cfg" | Where-Object { $_.Name -notmatch '^VEN_0000&DEV_0000' })
    if ($gpuCfgs.Count -eq 0) {
        Write-Log "Nenhum arquivo de perfil de GPU (VEN_*.cfg) encontrado em $($ab.ProfilesDir). Abra o MSI Afterburner 1 vez." "Yellow"
        return $false
    }

    $markerFile = Join-Path $ab.ProfilesDir ".modojogo_profiles_v5"
    if ((Test-Path $markerFile) -and (-not $ForceOverwrite)) {
        return $true
    }

    if (-not (Test-IsAdmin)) {
        Write-Log "Permissao de Administrador necessaria para gravar perfis em $($ab.ProfilesDir)." "Yellow"
        return $false
    }

    $wasRunning = ($null -ne (Get-Process -Name "MSIAfterburner" -ErrorAction SilentlyContinue))
    if ($wasRunning) {
        Stop-Process -Name "MSIAfterburner" -Force
        Start-Sleep -Milliseconds 1200
    }

    $plan       = Get-GpuTuningPlan -Hw $Hw
    $ecoCoreKHz = $plan.EcoBoostMHz * 1000
    $ecoFanMode = if ($Hw.IsLaptop) { 0 } else { 1 }
    $ecoFanPct  = if ($Hw.IsLaptop) { 35 } else { 30 }

    foreach ($cfg in $gpuCfgs) {
        $bak = "$($cfg.FullName).bak_modojogo"
        if (-not (Test-Path $bak)) { Copy-Item $cfg.FullName $bak -Force }

        $defaultsCurveHex = Get-IniSectionValue -FilePath $cfg.FullName -SectionName "Defaults" -KeyName "VFCurve"
        $summaryParts = [System.Collections.Generic.List[string]]::new()
        $safeStartupLines = $null

        foreach ($tierKey in @('Leve','Medio','Boost')) {
            $t = $plan.Tiers[$tierKey]
            $curve = $null
            if ($defaultsCurveHex -and $plan.IsDedicated) {
                $curve = New-UndervoltOverclockVFCurve -DefaultsHex $defaultsCurveHex -CoreBoostMHz $t.CoreMHz -VoltCapFrac $t.VoltFrac -PeakExtraMHz $t.PeakExtra -MatchPeakClk $true
            }
            $lines = [System.Collections.Generic.List[string]]@(
                "Format=2",
                "PowerLimit=$($t.PowerLimit)",
                "ThermalLimit=$($t.ThermalLimit)",
                "ThermalPrioritize=0",
                "CoreClkBoost=$($t.CoreMHz * 1000)",
                "MemClkBoost=$($t.MemMHz * 1000)"
            )
            if ($curve) { $lines.Add("VFCurve=$($curve.Hex)") }
            $lines.Add("FanMode=$($t.FanMode)")
            $lines.Add("FanSpeed=$($t.FanSpeed)")

            Set-IniSection -FilePath $cfg.FullName -SectionName "Profile$($t.Slot)" -KeyValueLines $lines.ToArray()
            if ($tierKey -eq 'Leve') { $safeStartupLines = $lines.ToArray() }
            $desc = if ($curve) { "$($t.Name)[S$($t.Slot)]: $($curve.PlateauMHz)MHz@$($curve.CapMv)mV (+$($t.CoreMHz)/+$($t.MemMHz))" } else { "$($t.Name)[S$($t.Slot)]: PL=$($t.PowerLimit)% (+$($t.CoreMHz)/+$($t.MemMHz))" }
            $summaryParts.Add($desc)
        }

        if ($safeStartupLines) {
            Set-IniSection -FilePath $cfg.FullName -SectionName "Startup" -KeyValueLines $safeStartupLines
        }

        $ecoCurve = $null
        if ($defaultsCurveHex -and $plan.IsDedicated) {
            $ecoCurve = New-UndervoltOverclockVFCurve -DefaultsHex $defaultsCurveHex -CoreBoostMHz $plan.EcoBoostMHz -VoltCapFrac $plan.EcoVoltFrac -PeakExtraMHz 0 -MatchPeakClk $false
        }
        $ecoLines = [System.Collections.Generic.List[string]]@(
            "Format=2",
            "PowerLimit=$($Hw.EcoPowerPct)",
            "ThermalLimit=75",
            "ThermalPrioritize=0",
            "CoreClkBoost=$ecoCoreKHz",
            "MemClkBoost=0"
        )
        if ($ecoCurve) { $ecoLines.Add("VFCurve=$($ecoCurve.Hex)") }
        $ecoLines.Add("FanMode=$ecoFanMode")
        $ecoLines.Add("FanSpeed=$ecoFanPct")
        Set-IniSection -FilePath $cfg.FullName -SectionName "Profile$EcoSlot" -KeyValueLines $ecoLines.ToArray()

        $ecoDesc = if ($ecoCurve) { "Eco[S$EcoSlot]: $($ecoCurve.PlateauMHz)MHz@$($ecoCurve.CapMv)mV PL=$($Hw.EcoPowerPct)%" } else { "Eco[S$EcoSlot]: PL=$($Hw.EcoPowerPct)%" }
        $summaryParts.Add($ecoDesc)
        Write-Log ("Perfis GPU gravados ($($cfg.Name)): " + ($summaryParts -join ' | ')) "Green"
    }

    foreach ($slot in @($SlotLeve, $SlotMedio, $SlotBoost, $EcoSlot)) {
        $slotFile = Join-Path $ab.ProfilesDir "Profile$slot.cfg"
        if (-not (Test-Path $slotFile)) {
            Set-Content -Path $slotFile -Value "[Settings]`r`nProfileContents=1" -Encoding Default -NoNewline
        }
    }

    Set-Content -Path $markerFile -Value ("Calibrado v5 (Leve/Medio/Boost/Eco + FSR/SDXL + ASPM Off) em " + (Get-Date -Format s) + " para " + $Hw.GpuName) -Encoding UTF8

    if ($wasRunning) {
        Start-Process -FilePath $ab.ExePath -ArgumentList "/s" -WindowStyle Hidden
        Start-Sleep -Seconds 2
    }
    return $true
}

function Invoke-AfterburnerProfile {
    param([int]$Slot)
    $ab = Find-Afterburner
    if (-not $ab) { return }
    $running = Get-Process -Name "MSIAfterburner" -ErrorAction SilentlyContinue
    if (-not $running) {
        Start-Process -FilePath $ab.ExePath -ArgumentList "/s" -WindowStyle Hidden
        Start-Sleep -Seconds 2
    }
    Start-Process -FilePath $ab.ExePath -ArgumentList "-Profile$Slot" -WindowStyle Hidden
}

function Optimize-RealtekAndAudio {
    param(
        [ValidateSet('Jogos','Streaming','Economia')]
        [string]$PresetMode = 'Jogos',
        [switch]$AllowRestartAudio,
        [switch]$Silent
    )
    if (-not $AutoOptimizeApps) { return }

    try {
        # 1. Estereo real + Nunca baixar som de jogos/OBS durante comunicacao
        $mmUser = 'HKCU:\Software\Microsoft\Multimedia\Audio'
        New-Item -Path $mmUser -Force | Out-Null
        Set-ItemProperty -Path $mmUser -Name 'AccessibilityMonoMixState' -Value 0 -Type DWord -Force
        Set-ItemProperty -Path $mmUser -Name 'UserDuckingPreference' -Value 3 -Type DWord -Force
    } catch {}

    if (Test-IsAdmin) {
        try {
            # 2. MMCSS Audio & Pro Audio para menor latencia e zero estalos sob carga
            $mmBase = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks'
            foreach ($taskN in @('Audio','Pro Audio')) {
                $tp = Join-Path $mmBase $taskN
                if (Test-Path $tp) {
                    $prioVal = if ($taskN -eq 'Pro Audio') { 2 } else { 6 }
                    Set-ItemProperty -Path $tp -Name 'Scheduling Category' -Value 'High' -Type String -Force
                    Set-ItemProperty -Path $tp -Name 'GPU Priority' -Value 8 -Type DWord -Force
                    Set-ItemProperty -Path $tp -Name 'Priority' -Value $prioVal -Type DWord -Force
                    if ($taskN -eq 'Pro Audio') {
                        Set-ItemProperty -Path $tp -Name 'Background Only' -Value 'False' -Type String -Force
                    }
                }
            }

            # 3. Impedir suspensao D3 de economia agressiva no chip Realtek (evita pops/atrasos)
            $audioClass = 'HKLM:\SYSTEM\CurrentControlSet\Control\Class\{4d36e96c-e325-11ce-bfc1-08002be10318}'
            Get-ChildItem $audioClass -ErrorAction SilentlyContinue | ForEach-Object {
                $drvProps = Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue
                if ($drvProps.DriverDesc -match 'Realtek') {
                    $pwrPath = Join-Path $_.PSPath 'PowerSettings'
                    if (Test-Path $pwrPath) {
                        $zero4 = [byte[]](0,0,0,0)
                        Set-ItemProperty -Path $pwrPath -Name 'ConservationIdleTime' -Value $zero4 -Type Binary -Force
                        Set-ItemProperty -Path $pwrPath -Name 'PerformanceIdleTime'  -Value $zero4 -Type Binary -Force
                        Set-ItemProperty -Path $pwrPath -Name 'IdlePowerState'       -Value $zero4 -Type Binary -Force
                    }
                }
            }

            # 4. Alinhar dispositivos Realtek ativos em 48000 Hz / 24-32bit quando estiverem em >=96kHz (evita resampling CPU para jogos/OBS)
            $fmtChanged = $false
            $stoppedAudio = $false
            try {
                if ($AllowRestartAudio) {
                    Stop-Service -Name 'Audiosrv' -Force -ErrorAction SilentlyContinue
                    $stoppedAudio = $true
                }
                foreach ($role in @('Render','Capture')) {
                    $mmRoot = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\MMDevices\Audio\$role"
                    Get-ChildItem $mmRoot -ErrorAction SilentlyContinue | ForEach-Object {
                        $epState = (Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue).DeviceState
                        if ($epState -ne 1) { return }
                        $prPath = Join-Path $_.PSPath 'Properties'
                        $fxPath = Join-Path $_.PSPath 'FxProperties'
                        $pr = Get-ItemProperty $prPath -ErrorAction SilentlyContinue
                        if (-not ($pr.'{b3f8fa53-0004-438e-9003-51a46e139bfc},6' -match 'Realtek')) { return }

                        foreach ($subPath in @($prPath, $fxPath)) {
                            if (-not (Test-Path $subPath)) { continue }
                            $obj = Get-ItemProperty $subPath -ErrorAction SilentlyContinue
                            if (-not $obj) { continue }
                            foreach ($prop in $obj.PSObject.Properties) {
                                if ($prop.Name -like 'PS*' -or -not ($prop.Value -is [byte[]])) { continue }
                                $blob = $prop.Value
                                if ($blob.Length -ge 40 -and $blob.Length -le 128) {
                                    $tag = [BitConverter]::ToUInt16($blob, 8)
                                    $ch  = [BitConverter]::ToUInt16($blob, 10)
                                    $sr  = [BitConverter]::ToUInt32($blob, 12)
                                    $ba  = [BitConverter]::ToUInt16($blob, 20)
                                    if ($tag -eq 0xFFFE -and ($ch -eq 1 -or $ch -eq 2) -and $sr -ge 96000 -and $ba -in @(2,4,6,8)) {
                                        $newBlob = [byte[]]::new($blob.Length)
                                        [Array]::Copy($blob, $newBlob, $blob.Length)
                                        [BitConverter]::GetBytes([uint32]48000).CopyTo($newBlob, 12)
                                        [BitConverter]::GetBytes([uint32](48000 * $ba)).CopyTo($newBlob, 16)
                                        Set-ItemProperty -Path $subPath -Name $prop.Name -Value $newBlob -Type Binary -Force -ErrorAction SilentlyContinue
                                        $fmtChanged = $true
                                    }
                                }
                            }
                        }
                    }
                }
            } finally {
                if ($stoppedAudio) {
                    Start-Service -Name 'Audiosrv' -ErrorAction SilentlyContinue
                }
            }

            if ($fmtChanged -and $AllowRestartAudio) {
                Write-Log "Realtek Audio padronizado para 48000 Hz / 24-32bit Estereo (sem perda e sem resampling de CPU)." "Green" -Silent:$Silent
            }
        } catch {}
    }

    # 5. Prioridade do AudioDg conforme workload
    $audPrio = if ($PresetMode -in @('Jogos','Streaming')) { [System.Diagnostics.ProcessPriorityClass]::AboveNormal } else { [System.Diagnostics.ProcessPriorityClass]::Normal }
    Get-Process -Name 'audiodg' -ErrorAction SilentlyContinue | ForEach-Object {
        try { if ($_.PriorityClass -ne $audPrio) { $_.PriorityClass = $audPrio } } catch {}
    }
}

function Get-PrimaryScreenResolution {
    try {
        Add-Type -AssemblyName System.Windows.Forms -ErrorAction SilentlyContinue
        $b = [System.Windows.Forms.Screen]::PrimaryScreen.Bounds
        if ($b.Width -ge 800 -and $b.Height -ge 600) { return @{ W = $b.Width; H = $b.Height } }
    } catch {}
    return @{ W = 1920; H = 1080 }
}

function Set-IniKeyValue {
    param([string]$FilePath, [string]$SectionName, [string]$Key, [string]$Value)
    $enc = New-Object System.Text.UTF8Encoding($false)
    $lines = [System.Collections.Generic.List[string]]::new()
    if (Test-Path $FilePath) {
        $lines.AddRange([string[]][System.IO.File]::ReadAllLines($FilePath, $enc))
    }
    $header = "[$SectionName]"
    $secIdx = -1; $endIdx = $lines.Count
    for ($i = 0; $i -lt $lines.Count; $i++) {
        if ($lines[$i].Trim() -ieq $header) {
            $secIdx = $i
            for ($j = $i + 1; $j -lt $lines.Count; $j++) {
                if ($lines[$j].Trim() -match '^\[.+\]$') { $endIdx = $j; break }
            }
            break
        }
    }
    if ($secIdx -lt 0) {
        $lines.Add($header)
        $lines.Add("$Key=$Value")
    } else {
        $found = $false
        for ($k = $secIdx + 1; $k -lt $endIdx; $k++) {
            if ($lines[$k] -match ("^" + [regex]::Escape($Key) + "\s*=")) {
                $lines[$k] = "$Key=$Value"
                $found = $true
                break
            }
        }
        if (-not $found) { $lines.Insert($endIdx, "$Key=$Value") }
    }
    [System.IO.File]::WriteAllLines($FilePath, $lines.ToArray(), $enc)
}

function Optimize-ObsStudio {
    param(
        $Hw,
        [ValidateSet('Jogos','Streaming')]
        [string]$PresetMode = 'Streaming',
        [switch]$ForceProfileUpdate,
        [switch]$Silent
    )
    if (-not $AutoOptimizeApps) { return }
    $obsRoot = Join-Path $env:APPDATA "obs-studio"
    if (-not (Test-Path $obsRoot)) { return }

    $obsRunning = ($null -ne (Get-Process -Name 'obs64','obs32' -ErrorAction SilentlyContinue))
    if ($obsRunning -and (-not $ForceProfileUpdate)) {
        Get-Process -Name 'obs64','obs32' -ErrorAction SilentlyContinue | ForEach-Object {
            try { if ($_.PriorityClass -ne [System.Diagnostics.ProcessPriorityClass]::High) { $_.PriorityClass = [System.Diagnostics.ProcessPriorityClass]::High } } catch {}
        }
        return
    }
    if ($obsRunning -and $ForceProfileUpdate) {
        Write-Log "OBS Studio esta aberto no momento; feche o OBS e execute novamente para gravar os arquivos sem conflito." "Yellow" -Silent:$Silent
        return
    }

    $globalIni = Join-Path $obsRoot "global.ini"
    if (Test-Path $globalIni) {
        if (-not (Test-Path "$globalIni.bak_modojogo")) { Copy-Item $globalIni "$globalIni.bak_modojogo" -Force }
        $hwBrowser = if ($Hw.LowVram -or $PresetMode -eq 'Jogos' -or $Hw.IsLaptop) { 'false' } else { 'true' }
        Set-IniKeyValue -FilePath $globalIni -SectionName 'General' -Key 'ProcessPriority' -Value 'High'
        Set-IniKeyValue -FilePath $globalIni -SectionName 'General' -Key 'BrowserHWAccel' -Value $hwBrowser
        Set-IniKeyValue -FilePath $globalIni -SectionName 'Audio'   -Key 'DisableAudioDucking' -Value 'true'
    }

    $scr = Get-PrimaryScreenResolution
    $baseW = $scr.W; $baseH = $scr.H
    if ($Hw.VramMB -ge 3500 -and $Hw.CpuCores -ge 4) {
        if ($baseW -gt 1920) { $outW = 1920; $outH = 1080 } else { $outW = [int]($baseW - ($baseW % 4)); $outH = [int]($baseH - ($baseH % 2)) }
    } else {
        $outW = 1280; $outH = 720
    }
    $fpsVal = if ($Hw.CpuCores -ge 4 -and $Hw.VramMB -ge 2000) { 60 } else { 30 }

    $streamEnc = 'obs_x264'; $recEnc = 'obs_x264'; $encKind = 'x264'
    if ($Hw.GpuVendor -eq 'NVIDIA' -and $Hw.VramMB -ge 2000) {
        $streamEnc = 'obs_nvenc_h264_tex'; $recEnc = 'obs_nvenc_h264_tex'; $encKind = 'nvenc'
    } elseif ($Hw.GpuVendor -eq 'AMD' -and $Hw.VramMB -ge 2000) {
        $streamEnc = 'h264_texture_amf'; $recEnc = 'h264_texture_amf'; $encKind = 'amf'
    } elseif ($Hw.GpuVendor -eq 'Intel') {
        $streamEnc = 'obs_qsv11'; $recEnc = 'obs_qsv11'; $encKind = 'qsv'
    }

    $streamBitrate = if ($outW -ge 1920 -and $fpsVal -eq 60) { 7500 } elseif ($Hw.VramMB -ge 3500) { 6000 } else { 4500 }
    $nvPreset = if ($PresetMode -eq 'Jogos' -or $Hw.IsLaptop -or $Hw.LowVram) { 'p4' } else { 'p5' }
    $cqpVal   = if ($PresetMode -eq 'Streaming' -and -not $Hw.IsLaptop) { 18 } else { 20 }

    $profilesRoot = Join-Path $obsRoot "basic\profiles"
    if (-not (Test-Path $profilesRoot)) { return }

    $targetDirs = [System.Collections.Generic.List[string]]::new()
    Get-ChildItem $profilesRoot -Directory -ErrorAction SilentlyContinue | ForEach-Object { $targetDirs.Add($_.FullName) }
    foreach ($named in @('Antigravity_Jogos_Desempenho','Antigravity_Streaming_Qualidade')) {
        $nd = Join-Path $profilesRoot $named
        if (-not (Test-Path $nd)) {
            $seed = $targetDirs | Select-Object -First 1
            if ($seed) { Copy-Item $seed $nd -Recurse -Force } else { New-Item $nd -ItemType Directory -Force | Out-Null }
        }
        if (-not $targetDirs.Contains($nd)) { $targetDirs.Add($nd) }
    }

    $utf8NoBom = New-Object System.Text.UTF8Encoding($false)
    foreach ($pDir in $targetDirs) {
        $pName = Split-Path $pDir -Leaf
        $isJogosProf = ($pName -eq 'Antigravity_Jogos_Desempenho') -or ($pName -notmatch 'Streaming' -and $PresetMode -eq 'Jogos')
        $localNvPreset = if ($isJogosProf -or $Hw.IsLaptop) { 'p4' } else { 'p5' }
        $localCqp      = if ($isJogosProf -or $Hw.IsLaptop) { 20 } else { 18 }

        $basicIni = Join-Path $pDir 'basic.ini'
        if ((Test-Path $basicIni) -and (-not (Test-Path "$basicIni.bak_modojogo"))) { Copy-Item $basicIni "$basicIni.bak_modojogo" -Force }
        if ($pName -eq 'Antigravity_Jogos_Desempenho') { Set-IniKeyValue -FilePath $basicIni -SectionName 'General' -Key 'Name' -Value 'Antigravity - Jogos (Desempenho)' }
        if ($pName -eq 'Antigravity_Streaming_Qualidade') { Set-IniKeyValue -FilePath $basicIni -SectionName 'General' -Key 'Name' -Value 'Antigravity - Streaming (Alta Qualidade)' }

        Set-IniKeyValue -FilePath $basicIni -SectionName 'Output' -Key 'Mode' -Value 'Advanced'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'BaseCX' -Value "$baseW"
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'BaseCY' -Value "$baseH"
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'OutputCX' -Value "$outW"
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'OutputCY' -Value "$outH"
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'FPSType' -Value '1'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'FPSCommon' -Value "$fpsVal"
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'FPSInt' -Value "$fpsVal"
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'ScaleType' -Value 'lanczos'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'ColorFormat' -Value 'NV12'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'ColorSpace' -Value '709'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Video'  -Key 'ColorRange' -Value 'Partial'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Audio'  -Key 'SampleRate' -Value '48000'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'Audio'  -Key 'ChannelSetup' -Value 'Stereo'

        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'Encoder' -Value $streamEnc
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'RecEncoder' -Value $recEnc
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'RecFormat2' -Value 'mkv'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'AudioEncoder' -Value 'ffmpeg_aac'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'RecTracks' -Value '3'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'Track1Bitrate' -Value '192'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'Track2Bitrate' -Value '320'
        Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'Track3Bitrate' -Value '320'

        $curRecPath = Get-IniSectionValue -FilePath $basicIni -SectionName 'AdvOut' -KeyName 'RecFilePath'
        if ([string]::IsNullOrWhiteSpace($curRecPath) -or (-not (Test-Path $curRecPath))) {
            $bestDrv = Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3" | Sort-Object FreeSpace -Descending | Select-Object -First 1
            if ($bestDrv) {
                $recDir = Join-Path ($bestDrv.DeviceID + "\") "Videos"
                New-Item -Path $recDir -ItemType Directory -Force | Out-Null
                Set-IniKeyValue -FilePath $basicIni -SectionName 'AdvOut' -Key 'RecFilePath' -Value ($recDir.Replace('\','/'))
            }
        }

        if ($encKind -eq 'nvenc') {
            $sObj = [ordered]@{ rate_control='CBR'; bitrate=$streamBitrate; preset2=$localNvPreset; tuning='hq'; multipass='qres'; profile='high'; bf=2; psycho_aq=$true; lookahead=$false }
            $rObj = [ordered]@{ rate_control='CQP'; cqp=$localCqp; preset2=$localNvPreset; tuning='hq'; multipass='qres'; profile='high'; bf=2; psycho_aq=$true; lookahead=$false }
        } elseif ($encKind -eq 'x264') {
            $xPreset = if ($Hw.CpuCores -ge 8) { 'faster' } elseif ($Hw.CpuCores -ge 6) { 'veryfast' } else { 'superfast' }
            $sObj = [ordered]@{ rate_control='CBR'; bitrate=$streamBitrate; preset=$xPreset; profile='high' }
            $rObj = [ordered]@{ rate_control='CRF'; crf=($localCqp + 1); preset=$xPreset; profile='high' }
        } else {
            $sObj = [ordered]@{ rate_control='CBR'; bitrate=$streamBitrate }
            $rObj = [ordered]@{ rate_control='CQP'; cqp=$localCqp }
        }

        $sJsonPath = Join-Path $pDir 'streamEncoder.json'
        $rJsonPath = Join-Path $pDir 'recordEncoder.json'
        if ((Test-Path $sJsonPath) -and (-not (Test-Path "$sJsonPath.bak_modojogo"))) { Copy-Item $sJsonPath "$sJsonPath.bak_modojogo" -Force }
        if ((Test-Path $rJsonPath) -and (-not (Test-Path "$rJsonPath.bak_modojogo"))) { Copy-Item $rJsonPath "$rJsonPath.bak_modojogo" -Force }
        [System.IO.File]::WriteAllText($sJsonPath, ($sObj | ConvertTo-Json -Compress), $utf8NoBom)
        [System.IO.File]::WriteAllText($rJsonPath, ($rObj | ConvertTo-Json -Compress), $utf8NoBom)
    }
    Write-Log "OBS Studio calibrado ($encKind ${outW}x${outH}@${fpsVal}fps | CQP $cqpVal / Stream ${streamBitrate}kbps | Audio 48kHz 192/320k)." "Green" -Silent:$Silent
}

function Optimize-OtherPcPrograms {
    param($Hw, [switch]$Silent)
    if (-not $AutoOptimizeApps) { return }

    # 1. Wallpaper Engine: pausar automaticamente VRAM/GPU em tela cheia / maximizado
    $wePaths = @()
    Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' } | ForEach-Object {
        foreach ($sub in @('Program Files (x86)\Steam\steamapps\common\wallpaper_engine\config.json','SteamLibrary\steamapps\common\wallpaper_engine\config.json')) {
            $p = Join-Path $_.Root $sub
            if (Test-Path $p) { $wePaths += $p }
        }
    }
    foreach ($we in $wePaths) {
        if ($null -eq (Get-Process -Name 'wallpaper32','wallpaper64' -ErrorAction SilentlyContinue)) {
            try {
                if (-not (Test-Path "$we.bak_modojogo")) { Copy-Item $we "$we.bak_modojogo" -Force }
                $txt = [System.IO.File]::ReadAllText($we, [System.Text.Encoding]::UTF8)
                $upd = $txt
                $upd = $upd -replace '("playbackmaximized"\s*:\s*)"[^"]+"', '$1"pause"'
                $upd = $upd -replace '("pausevram"\s*:\s*)false', '$1true'
                if ($Hw.LowVram -or $Hw.IsLaptop) {
                    $upd = $upd -replace '("fps"\s*:\s*)\d+', '$130'
                }
                if ($upd -ne $txt) {
                    [System.IO.File]::WriteAllText($we, $upd, (New-Object System.Text.UTF8Encoding($false)))
                    Write-Log "Wallpaper Engine configurado para liberar GPU/VRAM durante jogos e OBS." "Green" -Silent:$Silent
                }
            } catch {}
        }
    }

    # 2. Discord: protege hardware em GPUs de menor VRAM e mantem audio limpo
    $discCfg = Join-Path $env:APPDATA "discord\settings.json"
    if ((Test-Path $discCfg) -and ($null -eq (Get-Process -Name 'Discord' -ErrorAction SilentlyContinue))) {
        try {
            if (-not (Test-Path "$discCfg.bak_modojogo")) { Copy-Item $discCfg "$discCfg.bak_modojogo" -Force }
            $dObj = Get-Content $discCfg -Raw -Encoding UTF8 | ConvertFrom-Json
            if ($Hw.VramMB -le 3500 -or $Hw.IsLaptop) {
                $dObj | Add-Member -NotePropertyName 'enableHardwareAcceleration' -NotePropertyValue $false -Force
            }
            ($dObj | ConvertTo-Json -Depth 6) | Set-Content $discCfg -Encoding UTF8
        } catch {}
    }

    # 3. Steam: desativar WebView GPU acceleration pesada em GPUs de 4GB ou notebook
    $steamReg = 'HKCU:\Software\Valve\Steam'
    if ((Test-Path $steamReg) -and ($Hw.LowVram -or $Hw.IsLaptop)) {
        Set-ItemProperty -Path $steamReg -Name 'GPUAccelWebViews' -Value 0 -Type DWord -ErrorAction SilentlyContinue
        Set-ItemProperty -Path $steamReg -Name 'GPUAccelWebViewsV3' -Value 0 -Type DWord -ErrorAction SilentlyContinue
    }
}

function Set-VdfKeyValue {
    param([string]$FilePath, [string]$Key, [string]$Value)
    if (-not (Test-Path $FilePath)) { return }
    $enc = New-Object System.Text.UTF8Encoding($false)
    $raw = [System.IO.File]::ReadAllText($FilePath, $enc)
    $esc = [regex]::Escape($Key)
    if ($raw -match ('"' + $esc + '"\s+"[^"]*"')) {
        $upd = [regex]::Replace($raw, '("' + $esc + '"\s+)"[^"]*"', "`${1}`"$Value`"")
    } else {
        $idxEnd = $raw.LastIndexOf('}')
        if ($idxEnd -gt 0) {
            $upd = $raw.Substring(0, $idxEnd) + "`t`"$Key`"`t`t`"$Value`"`r`n}" + $raw.Substring($idxEnd + 1)
        } else {
            $upd = $raw
        }
    }
    if ($upd -ne $raw) { [System.IO.File]::WriteAllText($FilePath, $upd, $enc) }
}

function Find-ExistingUpscalers {
    $found = [ordered]@{
        LosslessScaling = $null
        Magpie          = $null
        OptiScaler      = $null
    }
    $drives = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' -and (Test-Path $_.Root) }
    foreach ($drv in $drives) {
        foreach ($sub in @('Program Files (x86)\Steam\steamapps\common\Lossless Scaling\LosslessScaling.exe','SteamLibrary\steamapps\common\Lossless Scaling\LosslessScaling.exe')) {
            $p = Join-Path $drv.Root $sub
            if (Test-Path $p) { $found.LosslessScaling = $p; break }
        }
    }
    $toolsDir = Join-Path $InstallDir "Tools"
    $magExe = Join-Path $toolsDir "Magpie\Magpie.exe"
    if (Test-Path $magExe) { $found.Magpie = $magExe }
    $optDir = Join-Path $toolsDir "OptiScaler"
    if ((Test-Path $optDir) -and (Get-ChildItem $optDir -Filter "*.dll" -ErrorAction SilentlyContinue).Count -gt 0) {
        $found.OptiScaler = $optDir
    }
    return [PSCustomObject]$found
}

function Optimize-AmdFsrAndUpscaling {
    param($Hw, [switch]$Silent)
    if (-not $Hw) { $Hw = Get-HardwareProfile }

    # 1. Windows Game Mode, FSE/Contrast e PCIe Link para DirectStorage/Upscaling sem gargalos
    try {
        $gb = 'HKCU:\Software\Microsoft\GameBar'
        New-Item -Path $gb -Force | Out-Null
        Set-ItemProperty -Path $gb -Name 'AllowAutoGameMode' -Value 1 -Type DWord -Force
        Set-ItemProperty -Path $gb -Name 'AutoGameModeEnabled' -Value 1 -Type DWord -Force
    } catch {}

    $calibrated = [System.Collections.Generic.List[string]]::new()

    # 2. CS2 (Source 2): calibra AMD FSR 1.0 + Reflex/Latencia conforme GPU/Monitor do PC
    if ($null -eq (Get-Process -Name 'cs2' -ErrorAction SilentlyContinue)) {
        $steamRoots = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
        try {
            $regSteam = (Get-ItemProperty 'HKCU:\Software\Valve\Steam' -ErrorAction SilentlyContinue).SteamPath
            if ($regSteam) { [void]$steamRoots.Add(($regSteam -replace '/','\')) }
        } catch {}
        foreach ($pf in @(${env:ProgramFiles(x86)}, $env:ProgramFiles)) {
            if ($pf) { [void]$steamRoots.Add((Join-Path $pf 'Steam')) }
        }
        $cs2Cfgs = @()
        foreach ($sr in $steamRoots) {
            $ud = Join-Path $sr 'userdata'
            if (Test-Path $ud) {
                $cs2Cfgs += Get-ChildItem -Path $ud -Filter 'cs2_video.txt' -Recurse -File -ErrorAction SilentlyContinue
            }
        }
        $cs2Cfgs = $cs2Cfgs | Sort-Object FullName -Unique
        foreach ($f in $cs2Cfgs) {
            if (-not (Test-Path "$($f.FullName).bak_modojogo")) { Copy-Item $f.FullName "$($f.FullName).bak_modojogo" -Force }
            $fsrMode = if ($Hw.LowVram) {
                if ($Hw.DisplayW -le 1440) { "1" } else { "2" }
            } elseif ($Hw.DisplayW -ge 2560 -and $Hw.VramMB -lt 10000) {
                "1"
            } else {
                "0"
            }
            $reflex = if ($Hw.GpuVendor -eq 'NVIDIA') { "2" } else { "1" }
            Set-VdfKeyValue -FilePath $f.FullName -Key 'setting.videocfg_fsr_detail' -Value $fsrMode
            Set-VdfKeyValue -FilePath $f.FullName -Key 'setting.r_low_latency' -Value $reflex
            Set-VdfKeyValue -FilePath $f.FullName -Key 'setting.mat_vsync' -Value '0'
            if ($Hw.LowVram) {
                Set-VdfKeyValue -FilePath $f.FullName -Key 'setting.gpu_mem_level' -Value '0'
            }
            $calibrated.Add("CS2 (FSR=$fsrMode, Reflex=$reflex)")
        }
    } else {
        Write-Log "CS2 aberto no momento; mantendo cs2_video.txt atual sem sobrescrever em uso." "Yellow" -Silent:$Silent
    }

    # 3. Overwatch: remove trava VSync 60Hz, ativa RenderScale dinamico em LowVRAM e HighTickInput
    $owIni = Join-Path ([Environment]::GetFolderPath('MyDocuments')) "Overwatch\Settings\Settings_v0.ini"
    if ((Test-Path $owIni) -and ($null -eq (Get-Process -Name 'Overwatch' -ErrorAction SilentlyContinue))) {
        if (-not (Test-Path "$owIni.bak_modojogo")) { Copy-Item $owIni "$owIni.bak_modojogo" -Force }
        Set-IniKeyValue -FilePath $owIni -SectionName 'Render.13' -Key 'VerticalSyncEnabled' -Value '"0"'
        Set-IniKeyValue -FilePath $owIni -SectionName 'Render.13' -Key 'TripleBufferingEnabled' -Value '"0"'
        Set-IniKeyValue -FilePath $owIni -SectionName 'Render.13' -Key 'DesiredFrameRate' -Value "`"$($Hw.DisplayHz)`""
        if ($Hw.LowVram) {
            Set-IniKeyValue -FilePath $owIni -SectionName 'Render.13' -Key 'AADetail' -Value '"1"'
            Set-IniKeyValue -FilePath $owIni -SectionName 'Render.13' -Key 'RenderScale' -Value '"85"'
        }
        Set-IniKeyValue -FilePath $owIni -SectionName 'Input.1' -Key 'HighTickInput' -Value '"1"'
        $calibrated.Add("Overwatch ($($Hw.DisplayHz)Hz sem VSync)")
    }

    # 4. Otimizacao global de cache DirectX/Shader e recomendacao de compatibilidade FSR
    $ups = Find-ExistingUpscalers
    $upInfo = @()
    if ($ups.LosslessScaling) { $upInfo += "LosslessScaling" }
    if ($ups.Magpie)          { $upInfo += "Magpie(FSR)" }
    if ($ups.OptiScaler)      { $upInfo += "OptiScaler(DLSS->FSR)" }
    $archNote = if ($Hw.NoTensorCores) { "GPU sem Tensor Cores -> AMD FSR 1/2/3 + OptiScaler/Magpie recomendado no lugar de DLSS" } else { "Tensor Cores OK (DLSS + FSR disponiveis)" }
    $calSummary = if ($calibrated.Count -gt 0) { $calibrated -join ', ' } else { "Config de jogos verificada" }
    $upText = if ($upInfo.Count -gt 0) { $upInfo -join '+' } else { "use [12] para instalar Magpie/OptiScaler" }
    Write-Log "AMD FSR & Upscaling: $calSummary | $archNote | Ferramentas: $upText" "Cyan" -Silent:$Silent
}

function Install-FsrToolchain {
    param($Hw)
    if (-not $Hw) { $Hw = Get-HardwareProfile }
    if (-not (Test-IsAdmin)) {
        Write-Log "Execute como Administrador para instalar ferramentas em $InstallDir." "Red"
        return
    }
    $toolsRoot = Join-Path $InstallDir "Tools"
    New-Item -Path $toolsRoot -ItemType Directory -Force | Out-Null

    try {
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    } catch {}

    # 1. Baixar Magpie (Open-Source AMD FSR 1.0 Universal para QUALQUER jogo em janela/borda)
    Write-Host "`n[1/2] Consultando versao oficial do Magpie (AMD FSR 1.0 Universal) no GitHub..." -ForegroundColor Cyan
    try {
        $relM = Invoke-RestMethod -Uri "https://api.github.com/repos/Blinue/Magpie/releases/latest" -Headers @{ 'User-Agent' = 'ModoJogoAutomatico' } -TimeoutSec 20
        $assetM = $relM.assets | Where-Object { $_.name -match 'x64.*\.zip$' } | Select-Object -First 1
        if (-not $assetM) { $assetM = $relM.assets | Where-Object { $_.name -match '\.zip$' } | Select-Object -First 1 }
        if ($assetM) {
            $zipM = Join-Path $env:TEMP $assetM.name
            $destM = Join-Path $toolsRoot "Magpie"
            Invoke-WebRequest -Uri $assetM.browser_download_url -OutFile $zipM -UseBasicParsing -TimeoutSec 90
            New-Item -Path $destM -ItemType Directory -Force | Out-Null
            Expand-Archive -Path $zipM -DestinationPath $destM -Force
            Remove-Item $zipM -Force -ErrorAction SilentlyContinue
            Write-Log "Magpie (AMD FSR 1.0 Universal) instalado em $destM" "Green"

            $batRunner = Join-Path ([Environment]::GetFolderPath('Desktop')) "Ativar_FSR_Universal_Magpie.bat"
            $runnerContent = "@echo off`r`nstart `"`" `"$destM\Magpie.exe`"`r`n"
            [System.IO.File]::WriteAllText($batRunner, $runnerContent, (New-Object System.Text.UTF8Encoding($false)))
            Write-Log "Atalho criado na Area de Trabalho: Ativar_FSR_Universal_Magpie.bat (abra o jogo em janela/borda e aperte Win+Shift+A)." "Cyan"
        }
    } catch {
        Write-Log "Nao foi possivel baixar Magpie agora ($($_.Exception.Message))." "Yellow"
    }

    # 2. Baixar OptiScaler (Converte DLSS/XeSS -> AMD FSR 2/3 para placas sem Tensor Cores ou DX11/DX12)
    Write-Host "`n[2/2] Consultando versao oficial do OptiScaler (Ponte DLSS -> AMD FSR 2/3) no GitHub..." -ForegroundColor Cyan
    try {
        $relO = Invoke-RestMethod -Uri "https://api.github.com/repos/optiscaler/OptiScaler/releases/latest" -Headers @{ 'User-Agent' = 'ModoJogoAutomatico' } -TimeoutSec 20
        $assetO = $relO.assets | Where-Object { $_.name -match '\.(zip|7z)$' } | Select-Object -First 1
        if ($assetO) {
            $arcO = Join-Path $env:TEMP $assetO.name
            $destO = Join-Path $toolsRoot "OptiScaler"
            Invoke-WebRequest -Uri $assetO.browser_download_url -OutFile $arcO -UseBasicParsing -TimeoutSec 90
            New-Item -Path $destO -ItemType Directory -Force | Out-Null
            if ($arcO -match '\.zip$') {
                Expand-Archive -Path $arcO -DestinationPath $destO -Force
            } elseif (Get-Command tar -ErrorAction SilentlyContinue) {
                & tar -xf $arcO -C $destO 2>$null
            }
            Remove-Item $arcO -Force -ErrorAction SilentlyContinue
            Write-Log "OptiScaler (DLSS/XeSS -> AMD FSR 2.x/3.x) extraido em $destO" "Green"

            # Detectar jogos com nvngx_dlss.dll e oferecer injecao segura com backup
            $dlssGames = [System.Collections.Generic.List[object]]::new()
            $drives = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' -and (Test-Path $_.Root) }
            foreach ($drv in $drives) {
                foreach ($sub in @('SteamLibrary\steamapps\common','Program Files (x86)\Steam\steamapps\common','Epic Games','Games','Jogos')) {
                    $base = Join-Path $drv.Root $sub
                    if (Test-Path $base) {
                        Get-ChildItem -Path $base -Filter "nvngx_dlss.dll" -Recurse -Depth 8 -File -ErrorAction SilentlyContinue | ForEach-Object {
                            $dlssGames.Add($_)
                        }
                    }
                }
            }
            if ($dlssGames.Count -gt 0) {
                Write-Host "`nJogos com suporte a DLSS encontrados (podem receber AMD FSR 2/3 via OptiScaler):" -ForegroundColor Cyan
                for ($idx = 0; $idx -lt $dlssGames.Count; $idx++) {
                    Write-Host (" [{0}] {1}" -f ($idx + 1), $dlssGames[$idx].DirectoryName)
                }
                $pick = Read-Host "Digite o numero do jogo para instalar OptiScaler (ou Enter para pular)"
                $nIdx = 0
                if ([int]::TryParse($pick, [ref]$nIdx) -and $nIdx -ge 1 -and $nIdx -le $dlssGames.Count) {
                    $targetDir = $dlssGames[$nIdx - 1].DirectoryName
                    $optMain = Get-ChildItem $destO -Filter "OptiScaler*.dll" -Recurse -File -ErrorAction SilentlyContinue | Select-Object -First 1
                    if (-not $optMain) { $optMain = Get-ChildItem $destO -Filter "nvngx.dll" -Recurse -File -ErrorAction SilentlyContinue | Select-Object -First 1 }
                    $optIni  = Get-ChildItem $destO -Filter "OptiScaler.ini" -Recurse -File -ErrorAction SilentlyContinue | Select-Object -First 1
                    if ($optMain) {
                        $outDll = Join-Path $targetDir "nvngx.dll"
                        if ((Test-Path $outDll) -and (-not (Test-Path "$outDll.bak_modojogo"))) { Copy-Item $outDll "$outDll.bak_modojogo" -Force }
                        Copy-Item $optMain.FullName $outDll -Force
                        if ($optIni) { Copy-Item $optIni.FullName (Join-Path $targetDir "OptiScaler.ini") -Force }
                        Write-Log "OptiScaler instalado em $targetDir (aperte Insert dentro do jogo para escolher FSR 2/3)." "Green"
                    }
                }
            }
        }
    } catch {
        Write-Log "Nao foi possivel baixar OptiScaler agora ($($_.Exception.Message))." "Yellow"
    }
}

function Optimize-SdxlAndAiEnvironment {
    param($Hw, [switch]$Silent)
    if (-not $Hw) { $Hw = Get-HardwareProfile }

    # 1. Variaveis globais CUDA / PyTorch otimizadas para memoria e carregamento rapido
    $allocConf = if ($Hw.LowVram) {
        'expandable_segments:True,garbage_collection_threshold:0.6,max_split_size_mb:128'
    } elseif ($Hw.VramMB -le 8500) {
        'expandable_segments:True,garbage_collection_threshold:0.8'
    } else {
        'expandable_segments:True'
    }

    try {
        [Environment]::SetEnvironmentVariable('CUDA_MODULE_LOADING', 'LAZY', 'User')
        [Environment]::SetEnvironmentVariable('SAFETENSORS_FAST_GPU', '1', 'User')
        [Environment]::SetEnvironmentVariable('PYTORCH_CUDA_ALLOC_CONF', $allocConf, 'User')
    } catch {}

    $sdxlArgs = if ($Hw.LowVram -and $Hw.IsPascalOrOlder) {
        '--lowvram --opt-sdp-attention --no-half-vae --disable-nan-check'
    } elseif ($Hw.LowVram) {
        '--lowvram --opt-sdp-attention'
    } elseif ($Hw.VramMB -le 8500) {
        '--medvram-sdxl --opt-sdp-attention'
    } else {
        '--opt-sdp-attention'
    }

    # 2. Detectar ComfyUI / Automatic1111 / Forge / Fooocus / SD.Next e gerar launcher SDXL sob medida
    $aiRoots = [System.Collections.Generic.List[string]]::new()
    $drives = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' -and (Test-Path $_.Root) }
    foreach ($drv in $drives) {
        Get-ChildItem -Path $drv.Root -Directory -Depth 2 -ErrorAction SilentlyContinue | Where-Object {
            $_.Name -match 'ComfyUI|stable-diffusion|webui|forge|Fooocus|SDNext|SD\.Next|InvokeAI'
        } | ForEach-Object { $aiRoots.Add($_.FullName) }
    }

    $configuredCount = 0
    $utf8NoBom = New-Object System.Text.UTF8Encoding($false)
    foreach ($dir in $aiRoots) {
        $webuiBat = Join-Path $dir "webui-user.bat"
        if (Test-Path $webuiBat) {
            if (-not (Test-Path "$webuiBat.bak_modojogo")) { Copy-Item $webuiBat "$webuiBat.bak_modojogo" -Force }
            $txt = [System.IO.File]::ReadAllText($webuiBat, $utf8NoBom)
            $upd = $txt -replace '(?m)^set\s+COMMANDLINE_ARGS=.*$', "set COMMANDLINE_ARGS=$sdxlArgs"
            if ($upd -notmatch 'PYTORCH_CUDA_ALLOC_CONF') {
                $upd = "set CUDA_MODULE_LOADING=LAZY`r`nset SAFETENSORS_FAST_GPU=1`r`nset PYTORCH_CUDA_ALLOC_CONF=$allocConf`r`n" + $upd
            }
            if ($upd -ne $txt) { [System.IO.File]::WriteAllText($webuiBat, $upd, $utf8NoBom) }
            $configuredCount++
        }
        $mainPy = Join-Path $dir "main.py"
        if ((Test-Path $mainPy) -and ($dir -match 'ComfyUI')) {
            $cFlags = if ($Hw.LowVram) { "--lowvram --preview-method auto" } elseif ($Hw.VramMB -le 8500) { "--normalvram --preview-method auto" } else { "--highvram --preview-method auto" }
            $comfyBat = Join-Path $dir "Start_SDXL_Optimized.bat"
            $cBody = "@echo off`r`nset CUDA_MODULE_LOADING=LAZY`r`nset SAFETENSORS_FAST_GPU=1`r`nset PYTORCH_CUDA_ALLOC_CONF=$allocConf`r`npython main.py $cFlags`r`npause`r`n"
            [System.IO.File]::WriteAllText($comfyBat, $cBody, $utf8NoBom)
            $configuredCount++
        }
    }

    $pfUsage = Get-CimInstance Win32_PageFileUsage -ErrorAction SilentlyContinue | Select-Object -First 1
    $pfNote = if ($pfUsage) { "PageFile=$($pfUsage.AllocatedBaseSize)MB" } else { "PageFile=Auto" }
    $aiNote = if ($configuredCount -gt 0) { "$configuredCount ambiente(s) SDXL otimizados ($sdxlArgs)" } else { "Variaveis CUDA/PyTorch prontas ($sdxlArgs)" }
    Write-Log "Otimizacao SDXL / IA aplicada: $aiNote | CUDA_MODULE_LOADING=LAZY | $pfNote" "Green" -Silent:$Silent
}

function Apply-AudioVisualPreset {
    param(
        [ValidateSet('Jogos','Streaming','Economia')]
        [string]$PresetMode = 'Jogos',
        $Hw,
        [switch]$FullCalibrate,
        [switch]$Silent
    )
    if (-not $Hw) { $Hw = Get-HardwareProfile }
    [void](Save-ModoJogoConfig -ActiveAVPreset $PresetMode)

    Optimize-RealtekAndAudio -PresetMode $PresetMode -AllowRestartAudio:$FullCalibrate -Silent:$Silent
    if ($PresetMode -eq 'Economia') {
        Ensure-DisplayContrastAndColor -Silent:$Silent
    } else {
        Ensure-DisplayContrastAndColor -SkipGammaRamp:(-not $FullCalibrate) -Silent:$Silent
    }
    if ($FullCalibrate -and $PresetMode -ne 'Economia') {
        Optimize-ObsStudio -Hw $Hw -PresetMode $PresetMode -ForceProfileUpdate:$FullCalibrate -Silent:$Silent
        Optimize-OtherPcPrograms -Hw $Hw -Silent:$Silent
        Optimize-AmdFsrAndUpscaling -Hw $Hw -Silent:$Silent
        Optimize-SdxlAndAiEnvironment -Hw $Hw -Silent:$Silent
    }
    Write-Log "Predefinicao Audio/Imagem [$PresetMode] aplicada (48kHz Estereo, Sem Ducking, RGB Full 0-255, Gama/DDC)." "Cyan" -Silent:$Silent
}

function Set-BackgroundAppsThrottled {
    param([bool]$Throttle, $Hw, [switch]$InteractiveClosePrompt, [string]$WorkloadKind = 'Game')
    $targetPrio = if ($Throttle) { [System.Diagnostics.ProcessPriorityClass]::Idle } else { [System.Diagnostics.ProcessPriorityClass]::Normal }

    if ($Throttle -and $InteractiveClosePrompt) {
        $openBrowsers = @(Get-Process -Name $BackgroundThrottleBrowsers -ErrorAction SilentlyContinue)
        if ($openBrowsers.Count -gt 0) {
            $mb = [math]::Round((($openBrowsers | Measure-Object WorkingSet64 -Sum).Sum) / 1MB, 0)
            $resp = Read-Host "Navegadores abertos estao usando ~${mb} MB de RAM (e VRAM). Deseja fechar agora antes de iniciar? [S/N]"
            if ($resp -match '^[sS]') {
                Stop-Process -Name $BackgroundThrottleBrowsers -Force -ErrorAction SilentlyContinue
                Write-Log "Navegadores fechados para liberar RAM e VRAM." "Green"
            }
        }
    } elseif ($Throttle -and $CloseBrowsersAuto -and $WorkloadKind -eq 'Game') {
        Stop-Process -Name $BackgroundThrottleBrowsers -Force -ErrorAction SilentlyContinue
    }

    foreach ($pName in ($BackgroundThrottleBrowsers + $BackgroundThrottleHelpers)) {
        Get-Process -Name $pName -ErrorAction SilentlyContinue | ForEach-Object {
            try { if ($_.PriorityClass -ne $targetPrio) { $_.PriorityClass = $targetPrio } } catch {}
        }
    }

    if ($Throttle -and $Hw -and ($Hw.LowVram -or $Hw.RamFreeGB -lt 5.5)) {
        Get-Process -Name 'DSATray' -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue
    }
}

function Apply-PerformanceMode {
    param(
        [ValidateSet('Leve','Medio','Boost','Alto')]
        [string]$Tier = 'Boost',
        $Hw,
        $Schemes,
        [ValidateSet('Jogos','Streaming')]
        [string]$AudioVisualPreset = 'Jogos',
        [switch]$Interactive,
        [switch]$Silent
    )
    if ($Tier -eq 'Alto') { $Tier = 'Boost' }
    if (-not $Hw) { $Hw = Get-HardwareProfile }
    if (-not $Schemes) { $Schemes = Get-PowerSchemeIds }
    Ensure-SafeDesktopPowerSettings -Hw $Hw
    if ($Schemes.PerfGuid) { powercfg /setactive $Schemes.PerfGuid | Out-Null }

    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
    $plan = Get-GpuTuningPlan -Hw $Hw
    $t = $plan.Tiers[$Tier]
    Invoke-AfterburnerProfile -Slot $t.Slot

    Apply-AudioVisualPreset -PresetMode $AudioVisualPreset -Hw $Hw -Silent:$Silent
    Set-BackgroundAppsThrottled -Throttle $true -Hw $Hw -InteractiveClosePrompt:$Interactive -WorkloadKind $(if ($AudioVisualPreset -eq 'Streaming') { 'Streaming' } else { 'Game' })
    Write-Log ("MODO DESEMPENHO [$($t.Name)] ATIVADO -> Afterburner Slot $($t.Slot) (PL $($t.PowerLimit)% | UV+OC Core +$($t.CoreMHz) / Mem +$($t.MemMHz) MHz) | Preset [$AudioVisualPreset].") "Green" -Silent:$Silent
}

function Apply-EconomyMode {
    param($Hw, $Schemes, [switch]$Silent)
    if (-not $Hw) { $Hw = Get-HardwareProfile }
    if (-not $Schemes) { $Schemes = Get-PowerSchemeIds }
    Ensure-SafeDesktopPowerSettings -Hw $Hw
    if ($Schemes.EcoGuid) { powercfg /setactive $Schemes.EcoGuid | Out-Null }

    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
    Invoke-AfterburnerProfile -Slot $EcoSlot
    Apply-AudioVisualPreset -PresetMode 'Economia' -Hw $Hw -Silent:$Silent
    Set-BackgroundAppsThrottled -Throttle $false -Hw $Hw
    Write-Log "MODO ECONOMIA ATIVADO -> Plano Economia + Afterburner Slot $EcoSlot (Undervolt Eficiencia | PL $($Hw.EcoPowerPct)%)." "Yellow" -Silent:$Silent
}

function Get-DiscoveredAppSets {
    $games = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($g in $KnownGameExes) { [void]$games.Add($g) }

    $ultra = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($u in $UltraHeavyGameExes) { [void]$ultra.Add($u) }

    $stream = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($s in $StreamingExes) { [void]$stream.Add($s) }

    $create = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($c in $CreationWorkExes) { [void]$create.Add($c) }

    $audioC = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($a in $AudioConsoleExes) { [void]$audioC.Add($a) }

    $aiW = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($ai in $AiWorkloadExes) { [void]$aiW.Add($ai) }

    $upS = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($up in $UpscalerExes) { [void]$upS.Add($up) }

    $ignored = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($ig in ($IgnoredExeNames + $BackgroundThrottleBrowsers + $BackgroundThrottleHelpers)) { [void]$ignored.Add($ig) }

    $libSubPaths = @(
        'SteamLibrary\steamapps\common',
        'Program Files (x86)\Steam\steamapps\common',
        'Program Files\Steam\steamapps\common',
        'Steam\steamapps\common',
        'Epic Games',
        'Riot Games',
        'XboxGames',
        'GOG Games',
        'EA Games',
        'Games',
        'Jogos'
    )

    $drives = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' -and (Test-Path $_.Root) }
    foreach ($drv in $drives) {
        foreach ($sub in $libSubPaths) {
            $base = Join-Path $drv.Root $sub
            if (Test-Path $base) {
                Get-ChildItem -Path $base -Filter "*.exe" -Depth 3 -File -ErrorAction SilentlyContinue | ForEach-Object {
                    $baseName = $_.BaseName
                    if ((-not $ignored.Contains($baseName)) -and (-not $stream.Contains($baseName)) -and (-not $create.Contains($baseName)) -and $baseName -notmatch 'unins|setup|redist|crash|helper|overlay|update|launcher') {
                        [void]$games.Add($baseName)
                    }
                }
            }
        }
    }

    return [PSCustomObject]@{
        Games        = $games
        UltraHeavy   = $ultra
        Streaming    = $stream
        Creation     = $create
        AiWorkloads  = $aiW
        Upscalers    = $upS
        AudioConsole = $audioC
        Ignored      = $ignored
    }
}

function Get-ActiveWorkloadState {
    param($AppSets, $Config)
    if (-not $AppSets) { return $null }
    if (-not $Config) { $Config = Get-ModoJogoConfig }

    $procs = Get-Process -ErrorAction SilentlyContinue | Where-Object { $_.WorkingSet64 -gt 35MB }
    $gameList    = [System.Collections.Generic.List[object]]::new()
    $streamList  = [System.Collections.Generic.List[object]]::new()
    $createList  = [System.Collections.Generic.List[object]]::new()
    $aiList      = [System.Collections.Generic.List[object]]::new()
    $upscaleList = [System.Collections.Generic.List[object]]::new()
    $audioList   = [System.Collections.Generic.List[object]]::new()
    $otherList   = [System.Collections.Generic.List[object]]::new()
    $hasUltra    = $false

    foreach ($p in $procs) {
        $n = $p.ProcessName
        if ($AppSets.Streaming.Contains($n)) {
            $streamList.Add($p)
        } elseif ($AppSets.Games.Contains($n) -and $p.WorkingSet64 -gt 80MB) {
            $gameList.Add($p)
            if ($AppSets.UltraHeavy.Contains($n)) { $hasUltra = $true }
        } elseif ($AppSets.AiWorkloads.Contains($n) -and (($n -notmatch '^python') -or $p.WorkingSet64 -gt 600MB)) {
            $aiList.Add($p)
        } elseif ($AppSets.Upscalers.Contains($n)) {
            $upscaleList.Add($p)
        } elseif ($AppSets.Creation.Contains($n) -and $p.WorkingSet64 -gt 90MB) {
            $createList.Add($p)
        } elseif ($AppSets.AudioConsole.Contains($n)) {
            $audioList.Add($p)
        } elseif ((-not $AppSets.Ignored.Contains($n)) -and $p.MainWindowHandle -ne [IntPtr]::Zero -and $p.WorkingSet64 -gt 550MB) {
            $otherList.Add($p)
        }
    }

    $hasGame   = ($gameList.Count -gt 0)
    $hasStream = ($streamList.Count -gt 0)
    $hasCreate = ($createList.Count -gt 0)
    $hasAi     = ($aiList.Count -gt 0)
    $hasOther  = ($otherList.Count -gt 0)
    $needsPerf = ($hasGame -or $hasStream -or $hasCreate -or $hasAi -or $hasOther)

    $avPreset = if ($hasStream -or $hasCreate) { 'Streaming' } elseif ($needsPerf) { 'Jogos' } else { 'Economia' }

    $tier = $null
    if ($needsPerf) {
        if ($Config.AutoPerfTier -eq 'Alto') { $tier = 'Boost' }
        elseif ($Config.AutoPerfTier -in @('Leve','Medio','Boost')) {
            $tier = $Config.AutoPerfTier
        } else {
            $heavy3DCreate = ($createList | Where-Object { $_.ProcessName -in @('blender','afterfx','resolve','adobe premiere pro','premiere pro') }).Count -gt 0
            if (($hasGame -and $hasStream) -or $hasUltra -or $hasAi -or $heavy3DCreate) {
                $tier = 'Boost'
            } elseif ($hasGame -or $hasStream -or $hasCreate) {
                $tier = 'Medio'
            } else {
                $tier = 'Leve'
            }
        }
    }

    $labels = [System.Collections.Generic.List[string]]::new()
    if ($hasGame)   { $labels.Add(($gameList | Select-Object -First 1).ProcessName) }
    if ($hasStream) { $labels.Add(($streamList | Select-Object -First 1).ProcessName) }
    if ($hasAi)     { $labels.Add("IA:" + ($aiList | Select-Object -First 1).ProcessName) }
    if ($hasCreate) { $labels.Add(($createList | Select-Object -First 1).ProcessName) }
    if ($hasOther -and -not ($hasGame -or $hasStream -or $hasCreate -or $hasAi)) { $labels.Add(($otherList | Select-Object -First 1).ProcessName) }

    return [PSCustomObject]@{
        NeedsPerf       = $needsPerf
        Tier            = $tier
        AVPreset        = $avPreset
        Label           = ($labels -join ' + ')
        GameProcs       = $gameList
        StreamProcs     = $streamList
        AiProcs         = $aiList
        UpscaleProcs    = $upscaleList
        CreateProcs     = $createList
        AudioProcs      = $audioList
        OtherHeavyProcs = $otherList
        HasAudioConsole = ($audioList.Count -gt 0)
    }
}

function Set-ActiveWorkloadsPriority {
    param($State)
    if (-not $State -or -not $State.NeedsPerf) { return }
    foreach ($p in ($State.GameProcs + $State.StreamProcs + $State.UpscaleProcs)) {
        try { if ($p.PriorityClass -ne [System.Diagnostics.ProcessPriorityClass]::High) { $p.PriorityClass = [System.Diagnostics.ProcessPriorityClass]::High } } catch {}
    }
    foreach ($p in ($State.CreateProcs + $State.AiProcs + $State.OtherHeavyProcs)) {
        try {
            if ($p.PriorityClass -eq [System.Diagnostics.ProcessPriorityClass]::Normal -or $p.PriorityClass -eq [System.Diagnostics.ProcessPriorityClass]::BelowNormal) {
                $p.PriorityClass = [System.Diagnostics.ProcessPriorityClass]::AboveNormal
            }
        } catch {}
    }
}

function Start-GameWatcher {
    param([switch]$Silent)
    $Hw      = Get-HardwareProfile
    $Schemes = Get-PowerSchemeIds
    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null

    $appSets           = Get-DiscoveredAppSets
    $lastScan          = Get-Date
    $lastStateKey      = ''
    $lastHeavySeen     = [DateTime]::MinValue
    $audioConsoleSeen  = $false

    $cfgInit = Get-ModoJogoConfig
    Write-Log ("Monitor automatico iniciado | GPU: $($Hw.GpuName) ($($Hw.VramMB) MB) | Modo Auto: $($cfgInit.AutoPerfTier) | Jogos/Apps indexados: $($appSets.Games.Count)") "Cyan" -Silent:$Silent

    while ($true) {
        if (((Get-Date) - $lastScan).TotalSeconds -ge 300) {
            $appSets  = Get-DiscoveredAppSets
            $lastScan = Get-Date
        }

        $cfg   = Get-ModoJogoConfig
        $state = Get-ActiveWorkloadState -AppSets $appSets -Config $cfg

        if ($state.HasAudioConsole -and -not $audioConsoleSeen) {
            Optimize-RealtekAndAudio -PresetMode $cfg.ActiveAVPreset -Silent:$Silent
            $audioConsoleSeen = $true
        } elseif (-not $state.HasAudioConsole) {
            $audioConsoleSeen = $false
        }

        if ($state.NeedsPerf) {
            $lastHeavySeen = Get-Date
            $targetKey = "Perf:$($state.Tier):$($state.AVPreset)"
            if ($lastStateKey -ne $targetKey) {
                Write-Log ("Carga detectada ($($state.Label)). Ativando Desempenho [$($state.Tier)] + Preset [$($state.AVPreset)]...") "Green" -Silent:$Silent
                Apply-PerformanceMode -Tier $state.Tier -Hw $Hw -Schemes $Schemes -AudioVisualPreset $state.AVPreset -Silent:$Silent
                $lastStateKey = $targetKey
            }
            Set-ActiveWorkloadsPriority -State $state
        } else {
            if ($lastStateKey -eq '') {
                Write-Log "Nenhum jogo ou programa pesado aberto. Aplicando MODO ECONOMIA inicial..." "Yellow" -Silent:$Silent
                Apply-EconomyMode -Hw $Hw -Schemes $Schemes -Silent:$Silent
                $lastStateKey = 'Eco'
            } elseif ($lastStateKey -ne 'Eco' -and (((Get-Date) - $lastHeavySeen).TotalSeconds -ge $EcoGraceSec)) {
                Write-Log "Programas pesados encerrados. Alternando para MODO ECONOMIA..." "Yellow" -Silent:$Silent
                Apply-EconomyMode -Hw $Hw -Schemes $Schemes -Silent:$Silent
                $lastStateKey = 'Eco'
            }
        }
        Start-Sleep -Seconds $PollIntervalSec
    }
}

function Install-AutoStartTask {
    if (-not (Test-IsAdmin)) {
        Write-Log "Execute como Administrador para instalar a tarefa agendada." "Red"
        return
    }
    New-Item -Path $InstallDir -ItemType Directory -Force | Out-Null
    $installedBat = Join-Path $InstallDir "Modo_Jogo_Automatico.bat"
    Copy-Item -Path $env:MODOJOGO_SELF -Destination $installedBat -Force

    $psPayload = "`$env:MODOJOGO_ARGS='-auto'; `$env:MODOJOGO_SELF='$installedBat'; `$l=[IO.File]::ReadAllLines('$installedBat',[Text.Encoding]::UTF8); `$i=[Array]::IndexOf(`$l,':: === INICIO_POWERSHELL ==='); `$c=`$l[(`$i+1)..(`$l.Length-1)] -join [Environment]::NewLine; & ([ScriptBlock]::Create(`$c))"
    $encoded   = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($psPayload))

    $currentUser = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
    $action      = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -EncodedCommand $encoded"
    $trigger     = New-ScheduledTaskTrigger -AtLogOn -User $currentUser
    $principal   = New-ScheduledTaskPrincipal -UserId $currentUser -LogonType Interactive -RunLevel Highest
    $settings  = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -ExecutionTimeLimit (New-TimeSpan -Days 0) -RestartCount 3 -RestartInterval (New-TimeSpan -Minutes 1) -MultipleInstances IgnoreNew

    Unregister-ScheduledTask -TaskName $TaskName -Confirm:$false -ErrorAction SilentlyContinue
    Register-ScheduledTask -TaskName $TaskName -Action $action -Trigger $trigger -Principal $principal -Settings $settings -Force | Out-Null
    Start-ScheduledTask -TaskName $TaskName

    Write-Log "Automacao instalada e iniciada em segundo plano (Tarefa Agendada: $TaskName)." "Green"
    Write-Log "Jogos, OBS, edicao e apps pesados alternam automaticamente entre Leve/Medio/Boost e Economia." "Gray"
}

function Restore-ModojogoBackupsInDir {
    param([string]$RootDir, [switch]$Recurse)
    if (-not $RootDir -or -not (Test-Path $RootDir)) { return }
    Get-ChildItem $RootDir -Filter "*.bak_modojogo" -Recurse:$Recurse -ErrorAction SilentlyContinue | ForEach-Object {
        $orig = $_.FullName -replace '\.bak_modojogo$', ''
        Copy-Item $_.FullName $orig -Force -ErrorAction SilentlyContinue
        Write-Log "Backup restaurado: $orig" "Gray"
    }
}

function Uninstall-AutoStartTask {
    if (-not (Test-IsAdmin)) {
        Write-Log "Execute como Administrador para desinstalar a automacao." "Red"
        return
    }
    Stop-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue
    Unregister-ScheduledTask -TaskName $TaskName -Confirm:$false -ErrorAction SilentlyContinue
    Get-CimInstance Win32_Process -Filter "Name='powershell.exe'" | Where-Object { $_.CommandLine -match 'ModoJogoAutomatico' } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force -ErrorAction SilentlyContinue }

    $ab = Find-Afterburner
    if ($ab -and (Test-Path $ab.ProfilesDir)) {
        foreach ($mk in @(".modojogo_profiles_v1", ".modojogo_profiles_v2", ".modojogo_profiles_v3", ".modojogo_profiles_v4", ".modojogo_profiles_v5")) {
            $markerFile = Join-Path $ab.ProfilesDir $mk
            if (Test-Path $markerFile) { Remove-Item $markerFile -Force }
        }
        Restore-ModojogoBackupsInDir -RootDir $ab.ProfilesDir
    }
    Restore-ModojogoBackupsInDir -RootDir (Join-Path $env:APPDATA "obs-studio") -Recurse
    Restore-ModojogoBackupsInDir -RootDir (Join-Path $env:APPDATA "discord")
    Restore-ModojogoBackupsInDir -RootDir (Join-Path ([Environment]::GetFolderPath('MyDocuments')) "Overwatch\Settings")
    foreach ($pf in @(${env:ProgramFiles(x86)}, $env:ProgramFiles)) {
        if ($pf) { Restore-ModojogoBackupsInDir -RootDir (Join-Path $pf "Steam\userdata") -Recurse }
    }
    Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' } | ForEach-Object {
        foreach ($sub in @('Program Files (x86)\Steam\steamapps\common\wallpaper_engine','SteamLibrary\steamapps\common\wallpaper_engine')) {
            Restore-ModojogoBackupsInDir -RootDir (Join-Path $_.Root $sub)
        }
    }

    if (Test-Path $InstallDir) { Remove-Item $InstallDir -Recurse -Force -ErrorAction SilentlyContinue }
    Write-Log "Automacao removida e backups originais restaurados." "Green"
}

function Show-Diagnostics {
    $Hw      = Get-HardwareProfile
    $Schemes = Get-PowerSchemeIds
    $ab      = Find-Afterburner
    $plan    = Get-GpuTuningPlan -Hw $Hw
    $cfg     = Get-ModoJogoConfig
    $appSets = Get-DiscoveredAppSets
    $ups     = Find-ExistingUpscalers
    $task    = Get-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue
    $mmUser  = Get-ItemProperty 'HKCU:\Software\Microsoft\Multimedia\Audio' -ErrorAction SilentlyContinue
    $obsProf = Join-Path $env:APPDATA 'obs-studio\basic\profiles'
    $cudaLaz = [Environment]::GetEnvironmentVariable('CUDA_MODULE_LOADING','User')
    $ptAlloc = [Environment]::GetEnvironmentVariable('PYTORCH_CUDA_ALLOC_CONF','User')

    Write-Host "`n=== DIAGNOSTICO DO HARDWARE, FSR, SDXL E AUTOMACAO ===" -ForegroundColor Cyan
    Write-Host (" CPU      : {0} ({1} nucleos / {2} threads)" -f $Hw.CpuName, $Hw.CpuCores, $Hw.CpuThreads)
    Write-Host (" RAM      : {0} GB livres / {1} GB totais" -f $Hw.RamFreeGB, $Hw.RamTotalGB)
    Write-Host (" Tipo PC  : {0} | Monitor: {1}x{2} @ {3}Hz" -f $(if ($Hw.IsLaptop) { "Notebook (Bateria)" } else { "Desktop" }), $Hw.DisplayW, $Hw.DisplayH, $Hw.DisplayHz)
    Write-Host (" GPU      : {0} ({1} | {2} MB VRAM{3})" -f $Hw.GpuName, $Hw.GpuVendor, $Hw.VramMB, $(if ($Hw.GpuTempC) { " | $($Hw.GpuTempC) C" } else { "" }))
    Write-Host (" Upscaler : {0}" -f $(if ($Hw.NoTensorCores) { "Sem Tensor Cores -> AMD FSR 1/2/3 + OptiScaler/Magpie recomendado" } else { "Tensor Cores + AMD FSR suportados" }))
    Write-Host (" Tools UP : Lossless={0} | Magpie={1} | OptiScaler={2}" -f $(if ($ups.LosslessScaling) {"OK"} else {"Nao"}), $(if ($ups.Magpie) {"OK"} else {"Nao"}), $(if ($ups.OptiScaler) {"OK"} else {"Nao"}))
    Write-Host (" SDXL Env : CUDA_MODULE_LOADING={0} | PYTORCH_CUDA_ALLOC_CONF={1}" -f $(if ($cudaLaz) {$cudaLaz} else {"(padrao)"}), $(if ($ptAlloc) {$ptAlloc} else {"(padrao)"}))
    foreach ($tk in @('Leve','Medio','Boost')) {
        $t = $plan.Tiers[$tk]
        Write-Host (" Perfil   : {0,-6} (Slot {1}) -> PL {2}% | Undervolt + OC (+{3} Core / +{4} Mem MHz)" -f $t.Name, $t.Slot, $t.PowerLimit, $t.CoreMHz, $t.MemMHz)
    }
    Write-Host (" Economia : Eco    (Slot {0}) -> PL {1}% | Undervolt Eficiencia (+{2} MHz base)" -f $EcoSlot, $Hw.EcoPowerPct, $plan.EcoBoostMHz)
    Write-Host (" Config   : Nivel Auto = {0} | Predefinicao Audio/Imagem = {1}" -f $cfg.AutoPerfTier, $cfg.ActiveAVPreset)
    Write-Host (" Energia  : GUID Desempenho = $($Schemes.PerfGuid) | GUID Economia = $($Schemes.EcoGuid)")
    Write-Host (" Afterb.  : {0}" -f $(if ($ab) { $ab.ExePath } else { "Nao encontrado" }))
    Write-Host (" OBS      : {0}" -f $(if (Test-Path $obsProf) { ((Get-ChildItem $obsProf -Directory | Select-Object -ExpandProperty Name) -join ', ') } else { "Nao instalado" }))
    Write-Host (" Audio    : MonoMix={0} | Ducking={1} (0=Estereo, 3=Sem reduzir volume)" -f $mmUser.AccessibilityMonoMixState, $mmUser.UserDuckingPreference)
    Write-Host (" Automacao: {0}" -f $(if ($task) { "$TaskName ($($task.State))" } else { "Nao instalada" }))
    Write-Host (" Apps DB  : {0} jogos | {1} streaming | {2} edicao/DAW | {3} IA/SDXL" -f $appSets.Games.Count, $appSets.Streaming.Count, $appSets.Creation.Count, $appSets.AiWorkloads.Count)
    foreach ($d in $Hw.Disks) {
        Write-Host (" Disco    : {0} [{1}, {2} GB]" -f $d.FriendlyName, $d.MediaType, $d.GB) -ForegroundColor Gray
    }
    Write-Host ""
}

function Repair-WindowsSystemErrors {
    param($Hw)
    if (-not $Hw) { $Hw = Get-HardwareProfile }
    if (-not (Test-IsAdmin)) {
        Write-Log "Execute como Administrador para reparar erros de registro, servicos e drivers do Windows." "Red"
        return
    }
    Write-Log "Iniciando verificacao e reparo de erros do Windows (ASPM, drivers orfaos, WMI, HVCI, DISM/SFC)..." "Cyan"
    Ensure-SafeDesktopPowerSettings -Hw $Hw

    # 1. Limpar Crash Dumps acumulados
    $freed = 0L
    foreach ($dp in @("$env:SystemRoot\LiveKernelReports", "$env:SystemRoot\Minidump", "$env:LOCALAPPDATA\CrashDumps")) {
        if (Test-Path $dp) {
            Get-ChildItem $dp -Filter '*.dmp' -Recurse -File -ErrorAction SilentlyContinue | ForEach-Object {
                $freed += $_.Length
                Remove-Item $_.FullName -Force -ErrorAction SilentlyContinue
            }
        }
    }
    if ($freed -gt 0) { Write-Log ("Dumps limpos: {0:N1} MB liberados." -f ($freed / 1MB)) "Gray" }

    # 2. Remover servicos de driver orfaos (ex. arquivo .sys inexistente em unidade existente)
    Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\*' -ErrorAction SilentlyContinue | ForEach-Object {
        $ip = $_.ImagePath
        $sn = $_.PSChildName
        if ($ip -and ($_.Type -eq 1 -or $_.Type -eq 2) -and ($_.Start -in @(0,1,2,3))) {
            $sysP = $null
            if ($ip -match '^(?:\\\?\?\\)?([A-Za-z]:\\.+?\.sys)$') { $sysP = $matches[1] }
            elseif ($ip -match '^(system32\\drivers\\.+\.sys)$') { $sysP = Join-Path $env:SystemRoot $matches[1] }
            if ($sysP -and ($sysP -match '^([A-Za-z]:)\\') -and (Test-Path ($matches[1] + '\')) -and (-not (Test-Path $sysP))) {
                sc.exe stop $sn 2>$null | Out-Null
                sc.exe delete $sn 2>$null | Out-Null
                Write-Log ("Driver orfao removido (SCM 7000): $sn ($sysP)" ) "Yellow"
            }
        }
    }

    # 3. Corrigir Autorecover MOFs quebrados e repositorio WMI
    try {
        $cimom = 'HKLM:\SOFTWARE\Microsoft\Wbem\CIMOM'
        $mofs = (Get-ItemProperty $cimom -ErrorAction SilentlyContinue).'Autorecover MOFs'
        if ($mofs) {
            $seen = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
            $clean = [System.Collections.Generic.List[string]]::new()
            $chg = $false
            foreach ($m in $mofs) {
                if ([string]::IsNullOrWhiteSpace($m)) { $chg = $true; continue }
                $exp = [Environment]::ExpandEnvironmentVariables($m)
                if ((Test-Path $exp) -and $seen.Add($m)) { $clean.Add($m) } else { $chg = $true }
            }
            $defMof = Get-ChildItem "$env:ProgramData\Microsoft\Windows Defender\Platform" -Filter 'ProtectionManagement.mof' -Recurse -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending | Select-Object -First 1
            if ($defMof -and $seen.Add($defMof.FullName)) { $clean.Add($defMof.FullName); $chg = $true }
            if ($chg) {
                Set-ItemProperty -Path $cimom -Name 'Autorecover MOFs' -Value ([string[]]$clean.ToArray()) -Type MultiString -Force
                if ($defMof) { mofcomp $defMof.FullName 2>$null | Out-Null }
                Write-Log "Lista Autorecover MOFs do WMI corrigida." "Gray"
            }
        }
        $wv = winmgmt /verifyrepository 2>&1
        if ($LASTEXITCODE -ne 0 -or ($wv -join ' ') -match 'inconsist|not consistent') {
            winmgmt /salvagerepository 2>$null | Out-Null
            Write-Log "Repositorio WMI reparado automaticamente." "Yellow"
        }
    } catch {}

    # 4. Resolver conflito HVCI (Kernel-Boot 124) se registrado no log recente
    try {
        $kb124 = Get-WinEvent -FilterHashtable @{ LogName='System'; ProviderName='Microsoft-Windows-Kernel-Boot'; Id=124; StartTime=(Get-Date).AddDays(-7) } -MaxEvents 1 -ErrorAction SilentlyContinue
        if ($kb124) {
            $hvci = 'HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity'
            if (Test-Path $hvci) {
                Set-ItemProperty -Path $hvci -Name 'Enabled' -Value 0 -Type DWord -Force
                Set-ItemProperty -Path $hvci -Name 'WasEnabledBy' -Value 0 -Type DWord -Force
            }
            $dg = 'HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard'
            if (Test-Path $dg) { Set-ItemProperty -Path $dg -Name 'EnableVirtualizationBasedSecurity' -Value 0 -Type DWord -Force }
            Write-Log "Conflito HVCI/VBS (Kernel-Boot 124) desativado." "Yellow"
        }
    } catch {}

    # 5. Instalar drivers PCI/Chipset oficiais pendentes via Windows Update caso haja falha de instalacao
    try {
        $badPci = Get-PnpDevice -ErrorAction SilentlyContinue | Where-Object { $_.Problem -eq 'CM_PROB_FAILED_INSTALL' -and $_.InstanceId -match '^PCI\\' }
        if ($badPci) {
            $sess = New-Object -ComObject Microsoft.Update.Session
            $srch = $sess.CreateUpdateSearcher()
            $res  = $srch.Search("IsInstalled=0 and Type='Driver'")
            $coll = New-Object -ComObject Microsoft.Update.UpdateColl
            foreach ($u in $res.Updates) {
                if ((($u.DriverHardwareID -join ';').ToLower()) -match 'pci\\ven_') { [void]$coll.Add($u) }
            }
            if ($coll.Count -gt 0) {
                $dl = $sess.CreateUpdateDownloader(); $dl.Updates = $coll; [void]$dl.Download()
                $in = $sess.CreateUpdateInstaller(); $in.Updates = $coll; [void]$in.Install()
                Write-Log ("Drivers oficiais de Chipset/PCI instalados via Windows Update: " + $coll.Count) "Green"
            }
        }
    } catch {}

    # 6. Agendar reparo offline seguro (sem travamento online) se houver Ntfs 55 recente
    try {
        $nt55 = Get-WinEvent -FilterHashtable @{ LogName='System'; ProviderName='Ntfs'; Id=55; StartTime=(Get-Date).AddDays(-7) } -MaxEvents 10 -ErrorAction SilentlyContinue
        if ($nt55) {
            $vols = $nt55 | ForEach-Object { if ($_.Properties.Count -gt 0) { ($_.Properties[0].Value + '').Trim() } } | Where-Object { $_ -match '^[A-Za-z]:$' } | Select-Object -Unique
            foreach ($v in $vols) {
                fsutil dirty set $v | Out-Null
                Write-Log ("Reparo NTFS offline seguro agendado para o proximo boot no volume $v.") "Yellow"
            }
        }
    } catch {}

    # 7. DISM + SFC no sistema
    $dismChk = DISM.exe /Online /Cleanup-Image /CheckHealth 2>&1
    if (($dismChk -join ' ') -match 'repar|repair|corrupt') {
        DISM.exe /Online /Cleanup-Image /RestoreHealth
    }
    sfc /scannow
    Write-Log "Verificacao e reparo de erros do Windows concluidos." "Green"
}

# === ROTEAMENTO DE EXECUCAO ===
$argNorm = ($env:MODOJOGO_ARGS + "").Trim().ToLower()
switch ($argNorm) {
    "-auto"         { Start-GameWatcher -Silent; exit 0 }
    "-monitor"      { Start-GameWatcher; exit 0 }
    "-leve"         { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-PerformanceMode -Tier 'Leve'  -Hw $Hw -Schemes $S; exit 0 }
    "-desempenho"   { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-PerformanceMode -Tier 'Boost' -Hw $Hw -Schemes $S; exit 0 }
    "-medio"        { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-PerformanceMode -Tier 'Medio' -Hw $Hw -Schemes $S; exit 0 }
    "-boost"        { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-PerformanceMode -Tier 'Boost' -Hw $Hw -Schemes $S; exit 0 }
    "-alto"         { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-PerformanceMode -Tier 'Boost' -Hw $Hw -Schemes $S; exit 0 }
    "-economia"     { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-EconomyMode -Hw $Hw -Schemes $S; exit 0 }
    "-presetjogos"  { $Hw = Get-HardwareProfile; Apply-AudioVisualPreset -PresetMode 'Jogos'     -Hw $Hw -FullCalibrate; exit 0 }
    "-presetstream" { $Hw = Get-HardwareProfile; Apply-AudioVisualPreset -PresetMode 'Streaming' -Hw $Hw -FullCalibrate; exit 0 }
    "-fsr"          { $Hw = Get-HardwareProfile; Optimize-AmdFsrAndUpscaling -Hw $Hw; exit 0 }
    "-instalarfsr"  { $Hw = Get-HardwareProfile; Optimize-AmdFsrAndUpscaling -Hw $Hw; Install-FsrToolchain -Hw $Hw; exit 0 }
    "-sdxl"         { $Hw = Get-HardwareProfile; Optimize-SdxlAndAiEnvironment -Hw $Hw; exit 0 }
    "-reparar"      { $Hw = Get-HardwareProfile; Repair-WindowsSystemErrors -Hw $Hw; exit 0 }
    "-calibrar"     {
        $Hw = Get-HardwareProfile
        $S  = Get-PowerSchemeIds
        Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
        Optimize-OtherPcPrograms -Hw $Hw
        Apply-PerformanceMode -Tier 'Boost' -Hw $Hw -Schemes $S -AudioVisualPreset 'Jogos'
        exit 0
    }
    "-corrigircor"  { Ensure-DisplayContrastAndColor; exit 0 }
}

$Hw      = Get-HardwareProfile
$Schemes = Get-PowerSchemeIds
$Plan    = Get-GpuTuningPlan -Hw $Hw

while ($true) {
    $cfgCur = Get-ModoJogoConfig
    $tL = $Plan.Tiers['Leve']; $tM = $Plan.Tiers['Medio']; $tB = $Plan.Tiers['Boost']
    Clear-Host
    Write-Host "================================================================================" -ForegroundColor Cyan
    Write-Host "  MODO JOGO AUTOMATICO - 4 PERFIS UV/OC + OBS + REALTEK + AMD FSR + SDXL" -ForegroundColor Cyan
    Write-Host "================================================================================" -ForegroundColor Cyan
    Write-Host (" Hardware : {0} | {1} GB RAM | {2} ({3} MB | {4}x{5}@{6}Hz)" -f $Hw.CpuName, $Hw.RamTotalGB, $Hw.GpuName, $Hw.VramMB, $Hw.DisplayW, $Hw.DisplayH, $Hw.DisplayHz) -ForegroundColor Gray
    Write-Host (" S$SlotLeve Leve  : +{0}/+{1} MHz (PL {2}%) | S$SlotMedio Medio: +{3}/+{4} (PL {5}%) | S$SlotBoost Boost: +{6}/+{7} (PL {8}%)" -f $tL.CoreMHz, $tL.MemMHz, $tL.PowerLimit, $tM.CoreMHz, $tM.MemMHz, $tM.PowerLimit, $tB.CoreMHz, $tB.MemMHz, $tB.PowerLimit) -ForegroundColor Gray
    Write-Host (" S$EcoSlot Eco   : +{0} base MHz (PL {1}%) | Auto: {2} | Preset: {3} | FSR: {4}" -f $Plan.EcoBoostMHz, $Hw.EcoPowerPct, $cfgCur.AutoPerfTier, $cfgCur.ActiveAVPreset, $(if ($Hw.NoTensorCores) {"FSR+OptiScaler"} else {"DLSS+FSR"})) -ForegroundColor Gray
    Write-Host "--------------------------------------------------------------------------------"
    Write-Host " [1] Calibrar Tudo (4 Perfis + OBS + Realtek + FSR/SDXL + Fix ASPM) & Ativar BOOST"
    Write-Host " [2] Ativar DESEMPENHO MEDIO     (Undervolt + Overclock Medio  | Slot $SlotMedio)"
    Write-Host " [3] Ativar DESEMPENHO BOOST     (Undervolt + Overclock Boost  | Slot $SlotBoost)"
    Write-Host " [4] Ativar MODO ECONOMIA        (Undervolt Eficiencia + Eco   | Slot $EcoSlot)"
    Write-Host " [5] Predefinicoes Audio/Imagem  (Jogos vs Streaming -> Realtek, OBS, Tela, Apps)"
    Write-Host " [6] Configurar Nivel Automatico (Auto Inteligente vs Fixo Leve/Medio/Boost)"
    Write-Host " [7] Iniciar Monitor Automatico nesta janela (jogos, OBS, IA/SDXL e apps)"
    Write-Host " [8] Instalar Automacao no Windows (roda oculto ao ligar o PC)"
    Write-Host " [9] Desinstalar Automacao e restaurar backups originais"
    Write-Host " [10] Corrigir Contraste / Filtro Branco (RGB Full 0-255, Gama 1.0 e DDC)"
    Write-Host " [11] Exibir Diagnostico Completo (Pecas, 4 Perfis, Audio, OBS, FSR e SDXL)"
    Write-Host " [12] AMD FSR Universal + Jogos  (Calibrar CS2/OW + Instalar Magpie & OptiScaler)"
    Write-Host " [13] Otimizacoes SDXL & IA      (CUDA Lazy, PyTorch Low-VRAM e Launchers WebUI)"
    Write-Host " [14] Reparar Erros do Windows   (ASPM, Drivers, WMI, HVCI 124, DISM/SFC, Boot)"
    Write-Host " [0] Sair"
    Write-Host ""
    $op = Read-Host "Escolha uma opcao"

    switch ($op) {
        "1" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Optimize-OtherPcPrograms -Hw $Hw
            Apply-AudioVisualPreset -PresetMode 'Jogos' -Hw $Hw -FullCalibrate
            Apply-PerformanceMode -Tier 'Boost' -Hw $Hw -Schemes $Schemes -AudioVisualPreset 'Jogos' -Interactive
            Pause
        }
        "2" {
            Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
            $av = Read-Host "Predefinicao [J] Jogos (padrao) ou [S] Streaming/Edicao"
            $avMode = if ($av -match '^[sS]') { 'Streaming' } else { 'Jogos' }
            Apply-PerformanceMode -Tier 'Medio' -Hw $Hw -Schemes $Schemes -AudioVisualPreset $avMode -Interactive
            Pause
        }
        "3" {
            Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
            $av = Read-Host "Predefinicao [J] Jogos (padrao) ou [S] Streaming/Edicao"
            $avMode = if ($av -match '^[sS]') { 'Streaming' } else { 'Jogos' }
            Apply-PerformanceMode -Tier 'Boost' -Hw $Hw -Schemes $Schemes -AudioVisualPreset $avMode -Interactive
            Pause
        }
        "4" {
            Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
            Apply-EconomyMode -Hw $Hw -Schemes $Schemes
            Pause
        }
        "5" {
            Write-Host "`n [1] Predefinicao JOGOS (Realtek 48kHz Estereo, OBS Baixa Latencia, RGB Full)" -ForegroundColor Cyan
            Write-Host " [2] Predefinicao STREAMING / EDICAO (Realtek 48kHz Estereo, OBS HQ + Audio 320k, RGB Full)" -ForegroundColor Cyan
            Write-Host " [3] Calibrar Todos os Programas apenas (OBS + Realtek + Discord + Wallpaper + Steam)" -ForegroundColor Cyan
            $sub = Read-Host "Escolha"
            if ($sub -eq "2") {
                Apply-AudioVisualPreset -PresetMode 'Streaming' -Hw $Hw -FullCalibrate
            } elseif ($sub -eq "3") {
                Optimize-RealtekAndAudio -PresetMode $cfgCur.ActiveAVPreset -AllowRestartAudio
                Optimize-ObsStudio -Hw $Hw -PresetMode $cfgCur.ActiveAVPreset -ForceProfileUpdate
                Optimize-OtherPcPrograms -Hw $Hw
            } else {
                Apply-AudioVisualPreset -PresetMode 'Jogos' -Hw $Hw -FullCalibrate
            }
            Pause
        }
        "6" {
            Write-Host "`n [A] Auto Inteligente (Leve=Apps pesados | Medio=Jogos/OBS | Boost=AAA/Live+IA/SDXL)" -ForegroundColor Cyan
            Write-Host " [L] Fixar automatico em LEVE  |  [M] Fixar em MEDIO  |  [B] Fixar em BOOST" -ForegroundColor Cyan
            $tChoice = (Read-Host "Escolha [A/L/M/B]").Trim().ToUpper()
            $newTier = switch ($tChoice) {
                'L' { 'Leve' }
                'M' { 'Medio' }
                'B' { 'Boost' }
                'T' { 'Boost' }
                default { 'Auto' }
            }
            [void](Save-ModoJogoConfig -AutoPerfTier $newTier)
            Write-Log "Nivel automatico salvo como: $newTier" "Green"
            Pause
        }
        "7" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Write-Host "`nMonitor ativo! Pressione Ctrl+C para encerrar ou feche esta janela.`n" -ForegroundColor Green
            Start-GameWatcher
        }
        "8" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Optimize-OtherPcPrograms -Hw $Hw
            Apply-AudioVisualPreset -PresetMode 'Jogos' -Hw $Hw -FullCalibrate
            Install-AutoStartTask
            Pause
        }
        "9" {
            Uninstall-AutoStartTask
            Pause
        }
        "10" {
            Ensure-DisplayContrastAndColor
            Write-Log "Verificacao de contraste, gama linear 1.0, RGB Full 0-255 e DDC concluida." "Green"
            Pause
        }
        "11" {
            Show-Diagnostics
            Pause
        }
        "12" {
            Optimize-AmdFsrAndUpscaling -Hw $Hw
            $ansFsr = Read-Host "`nDeseja baixar/instalar agora Magpie (FSR 1.0 Universal) e OptiScaler (DLSS->FSR 2/3) do GitHub oficial? [S/N]"
            if ($ansFsr -match '^[sS]') { Install-FsrToolchain -Hw $Hw }
            Pause
        }
        "13" {
            Optimize-SdxlAndAiEnvironment -Hw $Hw
            Pause
        }
        "14" {
            Repair-WindowsSystemErrors -Hw $Hw
            Pause
        }
        "0" { exit 0 }
    }
}
```