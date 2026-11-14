# Modo_Jogo_Automatico.bat — Perfis MSI Afterburner (Undervolt + Leve Overclock), Contraste e Energia por Hardware (Criado por Antigravity)

> **Nota de Autoria:** Este script `.bat` universal foi criado por **Antigravity** (assistente de IA do Google DeepMind) com foco em automação segura baseada no hardware real de cada máquina, sem depender de configurações fixas de um computador específico.

## O que este script faz

O `Modo_Jogo_Automatico.bat` é um utilitário universal (Batch + PowerShell nativo do Windows 10/11) que **inspeciona as peças atuais do PC** (CPU, memória RAM livre/total, placa de vídeo dedicada/integrada, VRAM, fabricante, temperatura, notebook vs. desktop, discos e planos de energia) e lê a curva V/F padrão (`[Defaults]`) do MSI Afterburner para:

1. **Criar automaticamente dois perfis calibrados (Undervolt + Overclock) no MSI Afterburner** (com backup automático `.bak_modojogo` antes de alterar qualquer `.cfg`):
   - **Slot 4 (`[Profile4]` — Modo Desempenho: Leve Overclock + Undervolt):**
     - Libera o `PowerLimit` máximo seguro suportado pela placa (via leitura em tempo real do `nvidia-smi` quando disponível em GPUs NVIDIA, ou padrão seguro `100%`), ajusta limite térmico seguro (`83 °C` em desktops / `80 °C` em notebooks) e mantém curva automática de ventoinhas.
     - Lê a curva padrão `[Defaults]` (`VFCurve=`) da GPU instalada, aplica **leve overclock seguro no núcleo** (ex.: `+40` a `+60 MHz` em desktops NVIDIA conforme VRAM / `+25 MHz` em notebooks / `+35 MHz` AMD dedicada) e **overclock conservador na memória VRAM** (`+150` a `+250 MHz` em desktops / `+100 MHz` em notebooks) **combinado com undervolt por achatamento de topo**: o script limita o teto de tensão de pico (~76–88% da rampa ativa) mantendo clock de pico igual ou ligeiramente acima do padrão de fábrica, reduzindo calor e evitando thermal/power throttling.
   - **Slot 5 (`[Profile5]` — Modo Economia: Undervolt de Eficiência):**
     - Reduz o `PowerLimit` ao mínimo estável detectado para a GPU (`65%` a `85%`), define limite térmico conservador (`75 °C`), zera offset de VRAM e aplica **undervolt de eficiência na curva V/F** (leve ganho na faixa baixa/média com teto de tensão limitado a ~58% da rampa ativa), mantendo uso leve, navegação e área de trabalho frios, responsivos e econômicos ao fechar os jogos.
2. **Proteção automática contra Filtro Branco / Contraste Lavado (`Ensure-DisplayContrastAndColor`):**
   - Garante **RGB Completo (0–255)** em displays NVIDIA via `nvapi64.dll` (evitando saída limitada `16-235` que deixa o preto acinzentado/lavado).
   - Restaura **Gama linear 1.0** (`SetDeviceGammaRamp`), desativa filtros de cor acidentais do Windows, mantém `GameDVR_FSEBehaviorMode = 0` e remove bloqueio `OverlayTestMode = 5` que prejudica composição/contraste em tela cheia.
   - Faz verificação segura via **DDC/CI (`dxva2.dll`)**: se algum monitor conectado estiver com **Contraste `< 30`** ou **Black Level RGB `> 55`**, restaura para valores nítidos padrão (`50`), sem sobrescrever monitores já bem calibrados.
3. **Alternar o Plano de Energia do Windows e processos em segundo plano:**
   - **Ao abrir jogos pesados (Modo Desempenho):** ativa **Desempenho Máximo** (ou **Alto Desempenho**, criando o esquema caso não exista), aplica o **Perfil 4** do MSI Afterburner e reduz para prioridade `Idle` processos que disputam CPU/RAM/VRAM (como navegadores, indexadores e utilitários secundários). Em sistemas com pouca VRAM (`<= 4200 MB`) ou pouca RAM livre, oferece fechamento opcional de navegadores ou encerra processos auxiliares ociosos.
   - **Ao fechar todos os jogos pesados (Modo Economia):** aguarda `12` segundos de tolerância para evitar trocas durante alt-tab ou reinício rápido de partida, volta o Windows para **Economia de Energia**, aplica o **Perfil 5** do MSI Afterburner, restaura a prioridade normal dos processos e valida contraste/gama do desktop.
4. **Automatização contínua em tempo real (CS2 e bibliotecas de jogos):**
   - Monitora automaticamente jogos pesados conhecidos (`cs2`, `valorant`, `fortniteclient-win64-shipping`, `r5apex`, `gta5`, `cyberpunk2077`, `eldenring`, `rdr2`, `cod`, etc.) **e** escaneia dinamicamente bibliotecas locais em todas as unidades (`SteamLibrary`, `Steam`, `Epic Games`, `Riot Games`, `XboxGames`, `GOG Games`, `EA Games`, `Games`, `Jogos`), ignorando instaladores, crash handlers e launchers.
   - Permite instalar uma **Tarefa Agendada oculta no Windows (`ModoJogoAutomatico_MSIAfterburner`)** que inicia silenciosamente ao fazer logon, sem janelas abertas na barra de tarefas.

---

## Como usar em qualquer PC

1. Baixe o arquivo [`Modo_Jogo_Automatico.bat`](./Modo_Jogo_Automatico.bat).
2. Clique com o botão direito e selecione **Executar como administrador** (ou dê duplo clique e aceite o UAC).
3. No menu interativo, escolha a opção desejada:
   - **`[1]` Calibrar Perfis (UV + OC) + Ativar MODO DESEMPENHO agora:** analisa o hardware, grava os Slots `4` e `5` no MSI Afterburner com undervolt + leve overclock e coloca o PC em desempenho máximo na hora.
   - **`[2]` Calibrar Perfis (UV) + Ativar MODO ECONOMIA agora:** alterna imediatamente para o perfil econômico com undervolt de eficiência.
   - **`[3]` Iniciar Monitor Automático nesta janela:** acompanha os jogos em tempo real exibindo o log na tela (`Ctrl+C` para encerrar).
   - **`[4]` Instalar Automação no Windows (segundo plano):** registra o monitor automático para rodar oculto sempre que o Windows iniciar.
   - **`[5]` Desinstalar Automação e restaurar backups:** remove a tarefa agendada e restaura os arquivos originais `.bak_modojogo` do MSI Afterburner.
   - **`[6]` Diagnóstico Completo:** mostra CPU, RAM, GPU, VRAM, calibração Undervolt/Overclock calculada, GUIDs de energia, discos e total de jogos detectados.
   - **`[7]` Corrigir Contraste / Filtro Branco agora:** força RGB Completo `0-255`, Gama linear `1.0`, otimizações de tela cheia limpas e checagem DDC/CI no monitor.

### Parâmetros de linha de comando (opcional)
- `Modo_Jogo_Automatico.bat -desempenho` — aplica Modo Desempenho diretamente e sai.
- `Modo_Jogo_Automatico.bat -economia` — aplica Modo Economia diretamente e sai.
- `Modo_Jogo_Automatico.bat -corrigircor` — aplica apenas a correção de contraste, gama e RGB Completo `0-255` e sai.
- `Modo_Jogo_Automatico.bat -monitor` — inicia o monitor automático com janela visível.
- `Modo_Jogo_Automatico.bat -auto` — inicia o monitor automático em modo silencioso (`%TEMP%\ModoJogoAutomatico.log`).

---

## Código Interno do `Modo_Jogo_Automatico.bat`

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
$PerfSlot          = 4       # Slot do MSI Afterburner reservado para DESEMPENHO (1 a 5)
$EcoSlot           = 5       # Slot do MSI Afterburner reservado para ECONOMIA (1 a 5)
$PerfCoreOffsetMHz = 0       # 0 = automatico por hardware (leve overclock + undervolt); ou valor em MHz (ex: 50)
$PerfMemOffsetMHz  = 0       # 0 = automatico por hardware (ex: +150/+200 MHz VRAM em GPU dedicada)
$AutoFixContrast   = $true   # $true corrige automaticamente filtro branco/contraste baixo (RGB Full 0-255, gama e DDC)
$PollIntervalSec   = 4       # Intervalo de verificacao do monitor automatico (segundos)
$EcoGraceSec       = 12      # Tempo apos fechar jogos antes de voltar para Economia (segundos)
$CloseBrowsersAuto = $false  # $true fecha navegadores ao abrir jogo em modo automatico; $false apenas reduz prioridade CPU
# ============================================================================

$TaskName    = "ModoJogoAutomatico_MSIAfterburner"
$InstallDir  = Join-Path $env:ProgramData "ModoJogoAutomatico"
$LogFile     = Join-Path $env:TEMP "ModoJogoAutomatico.log"

$KnownGameExes = @(
    'cs2','csgo','valorant','valorant-win64-shipping','fortniteclient-win64-shipping',
    'overwatch','r5apex','gta5','playgtav','gtaiv','cyberpunk2077','witcher3',
    'eldenring','rdr2','bf4','bf1','bfv','bf2042','forzahorizon4','forzahorizon5',
    'tslgame','dota2','league of legends','rainbowsix','escapefromtarkov',
    'helldivers2','once_human','acs','metro','eurotrucks2','farcry3','fc3_d3d11',
    'cod','bg3','bg3_dx11','starfield','hogwartslegacy','palworld','rustclient',
    'dayz_x64','hl2','rocketleague','marvelrivals','deadlock','apex','destiny2',
    'pubg','warframe.x64','re4','re5dx9','isaac-ng','repo','hades','oriwotw'
)

$IgnoredExeNames = @(
    'unins000','uninstall','setup','installer','vc_redist','dxsetup','crashhandler',
    'crashreporter','updater','launcher','bootstrap','anticheat','easyanticheat_eos_setup',
    'battleye','vgc','vgtray','steam','steamwebhelper','steamservice','epicwebhelper',
    'brave','chrome','msedge','firefox','opera','discord','obs64','rtss','msiafterburner'
)

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

    return [PSCustomObject]@{
        CpuName       = ($cpuObj.Name -replace '\s+', ' ').Trim()
        CpuCores      = $cpuObj.NumberOfCores
        CpuThreads    = $cpuObj.NumberOfLogicalProcessors
        RamTotalGB    = $ramTotalGB
        RamFreeGB     = $ramFreeGB
        IsLaptop      = $isLaptop
        GpuName       = $gpuMain.Name
        GpuVendor     = $vendor
        VramMB        = $vramMB
        LowVram       = ($vramMB -gt 0 -and $vramMB -le 4200)
        PerfPowerPct  = $nvMaxPct
        EcoPowerPct   = $nvMinPct
        GpuTempC      = $nvTempC
        Disks         = $disks
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
    $perfCore = 0; $perfMem = 0; $ecoBoost = 0; $perfPeakExtra = 0
    $perfVoltFrac = 0.84; $ecoVoltFrac = 0.58

    if ($isDedicated) {
        if ($Hw.IsLaptop) {
            $perfCore = 25; $perfMem = 100; $ecoBoost = 15; $perfPeakExtra = 0; $perfVoltFrac = 0.80; $ecoVoltFrac = 0.56
        } elseif ($Hw.GpuVendor -eq 'NVIDIA') {
            if ($Hw.VramMB -ge 6000) {
                $perfCore = 60; $perfMem = 250; $ecoBoost = 25; $perfPeakExtra = 15; $perfVoltFrac = 0.84; $ecoVoltFrac = 0.58
            } elseif ($Hw.VramMB -ge 3500) {
                $perfCore = 50; $perfMem = 200; $ecoBoost = 25; $perfPeakExtra = 12; $perfVoltFrac = 0.84; $ecoVoltFrac = 0.58
            } else {
                $perfCore = 40; $perfMem = 150; $ecoBoost = 20; $perfPeakExtra = 10; $perfVoltFrac = 0.84; $ecoVoltFrac = 0.58
            }
        } else {
            $perfCore = 35; $perfMem = 150; $ecoBoost = 15; $perfPeakExtra = 0; $perfVoltFrac = 0.85; $ecoVoltFrac = 0.60
        }
    }

    if ($PerfCoreOffsetMHz -gt 0) { $perfCore = $PerfCoreOffsetMHz }
    if ($PerfMemOffsetMHz  -gt 0) { $perfMem  = $PerfMemOffsetMHz }

    return [PSCustomObject]@{
        IsDedicated   = $isDedicated
        PerfCoreMHz   = $perfCore
        PerfMemMHz    = $perfMem
        PerfVoltFrac  = $perfVoltFrac
        PerfPeakExtra = $perfPeakExtra
        EcoBoostMHz   = $ecoBoost
        EcoVoltFrac   = $ecoVoltFrac
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
        $maxIdx = [math]::Max($minIdx, $peakIdx - 3)
        if ($foundIdx -ge 0) {
            $capIdx = [math]::Max($minIdx, [math]::Min($maxIdx, $foundIdx))
        } else {
            $capIdx = $maxIdx
        }
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
        Write-Log "MSI Afterburner nao encontrado; usando apenas gerenciamento de energia/processos do Windows." "Yellow"
        return $false
    }

    $gpuCfgs = @(Get-ChildItem $ab.ProfilesDir -Filter "VEN_*.cfg" | Where-Object { $_.Name -notmatch '^VEN_0000&DEV_0000' })
    if ($gpuCfgs.Count -eq 0) {
        Write-Log "Nenhum arquivo de perfil de GPU (VEN_*.cfg) encontrado em $($ab.ProfilesDir). Abra o MSI Afterburner 1 vez." "Yellow"
        return $false
    }

    $markerFile = Join-Path $ab.ProfilesDir ".modojogo_profiles_v2"
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

    $plan        = Get-GpuTuningPlan -Hw $Hw
    $perfCoreKHz = $plan.PerfCoreMHz * 1000
    $perfMemKHz  = $plan.PerfMemMHz * 1000
    $ecoCoreKHz  = $plan.EcoBoostMHz * 1000
    $perfTherm   = if ($Hw.IsLaptop) { 80 } else { 83 }
    $ecoFanMode  = if ($Hw.IsLaptop) { 0 } else { 1 }
    $ecoFanPct   = if ($Hw.IsLaptop) { 35 } else { 30 }

    foreach ($cfg in $gpuCfgs) {
        $bak = "$($cfg.FullName).bak_modojogo"
        if (-not (Test-Path $bak)) { Copy-Item $cfg.FullName $bak -Force }

        $defaultsCurveHex = Get-IniSectionValue -FilePath $cfg.FullName -SectionName "Defaults" -KeyName "VFCurve"
        $perfCurve = $null; $ecoCurve = $null
        if ($defaultsCurveHex -and $plan.IsDedicated) {
            $perfCurve = New-UndervoltOverclockVFCurve -DefaultsHex $defaultsCurveHex -CoreBoostMHz $plan.PerfCoreMHz -VoltCapFrac $plan.PerfVoltFrac -PeakExtraMHz $plan.PerfPeakExtra -MatchPeakClk $true
            $ecoCurve  = New-UndervoltOverclockVFCurve -DefaultsHex $defaultsCurveHex -CoreBoostMHz $plan.EcoBoostMHz -VoltCapFrac $plan.EcoVoltFrac  -PeakExtraMHz 0 -MatchPeakClk $false
        }

        $perfLines = [System.Collections.Generic.List[string]]@(
            "Format=2",
            "PowerLimit=$($Hw.PerfPowerPct)",
            "ThermalLimit=$perfTherm",
            "ThermalPrioritize=0",
            "CoreClkBoost=$perfCoreKHz",
            "MemClkBoost=$perfMemKHz"
        )
        if ($perfCurve) { $perfLines.Add("VFCurve=$($perfCurve.Hex)") }
        $perfLines.Add("FanMode=0")
        $perfLines.Add("FanSpeed=60")

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

        Set-IniSection -FilePath $cfg.FullName -SectionName "Profile$PerfSlot" -KeyValueLines $perfLines.ToArray()
        Set-IniSection -FilePath $cfg.FullName -SectionName "Profile$EcoSlot"  -KeyValueLines $ecoLines.ToArray()

        $perfMsg = if ($perfCurve) { "UV+OC $($perfCurve.PlateauMHz)MHz @ $($perfCurve.CapMv)mV (+$($plan.PerfCoreMHz) Core / +$($plan.PerfMemMHz) Mem)" } else { "PL=$($Hw.PerfPowerPct)% Core=+$($plan.PerfCoreMHz) Mem=+$($plan.PerfMemMHz)" }
        $ecoMsg  = if ($ecoCurve)  { "UV $($ecoCurve.PlateauMHz)MHz @ $($ecoCurve.CapMv)mV (PL=$($Hw.EcoPowerPct)%)" } else { "PL=$($Hw.EcoPowerPct)%" }
        Write-Log ("Perfis calibrados ($($cfg.Name)): Slot $PerfSlot [Desempenho: $perfMsg] | Slot $EcoSlot [Economia: $ecoMsg]") "Green"
    }

    foreach ($slot in @($PerfSlot, $EcoSlot)) {
        $slotFile = Join-Path $ab.ProfilesDir "Profile$slot.cfg"
        if (-not (Test-Path $slotFile)) {
            Set-Content -Path $slotFile -Value "[Settings]`r`nProfileContents=1" -Encoding Default -NoNewline
        }
    }

    Set-Content -Path $markerFile -Value ("Calibrado v2 em " + (Get-Date -Format s) + " para " + $Hw.GpuName) -Encoding UTF8

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

function Set-BackgroundAppsThrottled {
    param([bool]$Throttle, $Hw, [switch]$InteractiveClosePrompt)
    $targetPrio = if ($Throttle) { [System.Diagnostics.ProcessPriorityClass]::Idle } else { [System.Diagnostics.ProcessPriorityClass]::Normal }
    $browsers   = @('brave','chrome','msedge','firefox','opera')
    $helpers    = @('steamwebhelper','SearchIndexer','DSATray')

    if ($Throttle -and $InteractiveClosePrompt) {
        $openBrowsers = @(Get-Process -Name $browsers -ErrorAction SilentlyContinue)
        if ($openBrowsers.Count -gt 0) {
            $mb = [math]::Round((($openBrowsers | Measure-Object WorkingSet64 -Sum).Sum) / 1MB, 0)
            $resp = Read-Host "Navegadores abertos estao usando ~${mb} MB de RAM (e VRAM). Deseja fechar agora antes de jogar? [S/N]"
            if ($resp -match '^[sS]') {
                Stop-Process -Name $browsers -Force -ErrorAction SilentlyContinue
                Write-Log "Navegadores fechados para liberar RAM e VRAM." "Green"
            }
        }
    } elseif ($Throttle -and $CloseBrowsersAuto) {
        Stop-Process -Name $browsers -Force -ErrorAction SilentlyContinue
    }

    foreach ($pName in ($browsers + $helpers)) {
        Get-Process -Name $pName -ErrorAction SilentlyContinue | ForEach-Object {
            try {
                if ($_.PriorityClass -ne $targetPrio) { $_.PriorityClass = $targetPrio }
            } catch {}
        }
    }

    if ($Throttle -and $Hw -and ($Hw.LowVram -or $Hw.RamFreeGB -lt 5.5)) {
        Get-Process -Name 'DSATray' -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue
    }
}

function Apply-PerformanceMode {
    param($Hw, $Schemes, [switch]$Interactive, [switch]$Silent)
    if ($Schemes.PerfGuid) {
        powercfg /setactive $Schemes.PerfGuid | Out-Null
    }
    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
    Invoke-AfterburnerProfile -Slot $PerfSlot
    Ensure-DisplayContrastAndColor -SkipGammaRamp -Silent:$Silent
    Set-BackgroundAppsThrottled -Throttle $true -Hw $Hw -InteractiveClosePrompt:$Interactive
    $plan = Get-GpuTuningPlan -Hw $Hw
    Write-Log ("MODO DESEMPENHO ATIVADO -> Plano Maximo + Afterburner Perfil $PerfSlot (PL $($Hw.PerfPowerPct)% | UV+OC Core +$($plan.PerfCoreMHz) MHz / Mem +$($plan.PerfMemMHz) MHz).") "Green" -Silent:$Silent
}

function Apply-EconomyMode {
    param($Hw, $Schemes, [switch]$Silent)
    if ($Schemes.EcoGuid) {
        powercfg /setactive $Schemes.EcoGuid | Out-Null
    }
    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
    Invoke-AfterburnerProfile -Slot $EcoSlot
    Ensure-DisplayContrastAndColor -Silent:$Silent
    Set-BackgroundAppsThrottled -Throttle $false -Hw $Hw
    Write-Log "MODO ECONOMIA ATIVADO -> Plano Economia + Afterburner Perfil $EcoSlot (Undervolt Eficiencia | PL $($Hw.EcoPowerPct)%)." "Yellow" -Silent:$Silent
}

function Get-DiscoveredGameExes {
    $discovered = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    foreach ($g in $KnownGameExes) { [void]$discovered.Add($g) }

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
                    if ($IgnoredExeNames -notcontains $baseName -and $baseName -notmatch 'unins|setup|redist|crash|helper|overlay|update|launcher') {
                        [void]$discovered.Add($baseName)
                    }
                }
            }
        }
    }
    return ,$discovered
}

function Get-ActiveHeavyGame {
    param([System.Collections.Generic.HashSet[string]]$GameSet)
    if (-not $GameSet) { return $null }
    $procs = Get-Process -ErrorAction SilentlyContinue | Where-Object { $_.WorkingSet64 -gt 80MB }
    foreach ($p in $procs) {
        if ($GameSet.Contains($p.ProcessName)) {
            return $p.ProcessName
        }
    }
    return $null
}

function Start-GameWatcher {
    param([switch]$Silent)
    $Hw      = Get-HardwareProfile
    $Schemes = Get-PowerSchemeIds
    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null

    $gameSet       = Get-DiscoveredGameExes
    $lastScan      = Get-Date
    $inPerfMode    = $null
    $lastGameSeen  = [DateTime]::MinValue

    Write-Log ("Monitor automatico iniciado | GPU: $($Hw.GpuName) ($($Hw.VramMB) MB) | CPU: $($Hw.CpuName) | Jogos indexados: $($gameSet.Count)") "Cyan" -Silent:$Silent

    while ($true) {
        if (((Get-Date) - $lastScan).TotalSeconds -ge 300) {
            $gameSet  = Get-DiscoveredGameExes
            $lastScan = Get-Date
        }

        $activeGame = Get-ActiveHeavyGame -GameSet $gameSet
        if ($activeGame) {
            $lastGameSeen = Get-Date
            if ($inPerfMode -ne $true) {
                Write-Log "Jogo pesado detectado ($activeGame). Alternando para MODO DESEMPENHO..." "Green" -Silent:$Silent
                Apply-PerformanceMode -Hw $Hw -Schemes $Schemes -Silent:$Silent
                $inPerfMode = $true
            }
        } else {
            if ($null -eq $inPerfMode) {
                Write-Log "Nenhum jogo pesado aberto na inicializacao. Aplicando MODO ECONOMIA inicial..." "Yellow" -Silent:$Silent
                Apply-EconomyMode -Hw $Hw -Schemes $Schemes -Silent:$Silent
                $inPerfMode = $false
            } elseif ($inPerfMode -eq $true -and (((Get-Date) - $lastGameSeen).TotalSeconds -ge $EcoGraceSec)) {
                Write-Log "Nenhum jogo pesado em execucao. Alternando para MODO ECONOMIA..." "Yellow" -Silent:$Silent
                Apply-EconomyMode -Hw $Hw -Schemes $Schemes -Silent:$Silent
                $inPerfMode = $false
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
    Write-Log "Sempre que voce abrir jogos pesados (como CS2), o PC ira para DESEMPENHO e voltara para ECONOMIA ao fechar." "Gray"
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
        foreach ($mk in @(".modojogo_profiles_v1", ".modojogo_profiles_v2")) {
            $markerFile = Join-Path $ab.ProfilesDir $mk
            if (Test-Path $markerFile) { Remove-Item $markerFile -Force }
        }
        Get-ChildItem $ab.ProfilesDir -Filter "*.bak_modojogo" | ForEach-Object {
            $orig = $_.FullName -replace '\.bak_modojogo$', ''
            Copy-Item $_.FullName $orig -Force
            Write-Log "Backup restaurado: $orig" "Gray"
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
    $games   = Get-DiscoveredGameExes
    $task    = Get-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue

    Write-Host "`n=== DIAGNOSTICO DO HARDWARE ATUAL ===" -ForegroundColor Cyan
    Write-Host (" CPU      : {0} ({1} nucleos / {2} threads)" -f $Hw.CpuName, $Hw.CpuCores, $Hw.CpuThreads)
    Write-Host (" RAM      : {0} GB livres / {1} GB totais" -f $Hw.RamFreeGB, $Hw.RamTotalGB)
    Write-Host (" Tipo PC  : {0}" -f $(if ($Hw.IsLaptop) { "Notebook (Bateria)" } else { "Desktop" }))
    Write-Host (" GPU      : {0} ({1} | {2} MB VRAM{3})" -f $Hw.GpuName, $Hw.GpuVendor, $Hw.VramMB, $(if ($Hw.GpuTempC) { " | $($Hw.GpuTempC) C" } else { "" }))
    Write-Host (" Calibr.  : Desempenho (Slot $PerfSlot) = PL {0}% + Undervolt + Leve OC (+{1} MHz Core / +{2} MHz Mem)" -f $Hw.PerfPowerPct, $plan.PerfCoreMHz, $plan.PerfMemMHz)
    Write-Host ("          : Economia   (Slot $EcoSlot) = PL {0}% + Undervolt Eficiencia (+{1} MHz curvatura base)" -f $Hw.EcoPowerPct, $plan.EcoBoostMHz)
    Write-Host (" Energia  : GUID Desempenho = $($Schemes.PerfGuid) | GUID Economia = $($Schemes.EcoGuid)")
    Write-Host (" Afterb.  : {0}" -f $(if ($ab) { $ab.ExePath } else { "Nao encontrado" }))
    Write-Host (" Automacao: {0}" -f $(if ($task) { "$TaskName ($($task.State))" } else { "Nao instalada" }))
    Write-Host (" Jogos DB : {0} executaveis detectados (lista + bibliotecas locais)" -f $games.Count)
    foreach ($d in $Hw.Disks) {
        Write-Host (" Disco    : {0} [{1}, {2} GB]" -f $d.FriendlyName, $d.MediaType, $d.GB) -ForegroundColor Gray
    }
    Write-Host ""
}

# === ROTEAMENTO DE EXECUCAO ===
$argNorm = ($env:MODOJOGO_ARGS + "").Trim().ToLower()
switch ($argNorm) {
    "-auto"       { Start-GameWatcher -Silent; exit 0 }
    "-monitor"    { Start-GameWatcher; exit 0 }
    "-desempenho" { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-PerformanceMode -Hw $Hw -Schemes $S; exit 0 }
    "-economia"   { $Hw = Get-HardwareProfile; $S = Get-PowerSchemeIds; Apply-EconomyMode -Hw $Hw -Schemes $S; exit 0 }
    "-corrigircor" { Ensure-DisplayContrastAndColor; exit 0 }
}

$Hw      = Get-HardwareProfile
$Schemes = Get-PowerSchemeIds
$Plan    = Get-GpuTuningPlan -Hw $Hw

while ($true) {
    Clear-Host
    Write-Host "============================================================================" -ForegroundColor Cyan
    Write-Host "     MODO JOGO AUTOMATICO (HARDWARE-AWARE + UNDERVOLT / LEVE OVERCLOCK)" -ForegroundColor Cyan
    Write-Host "============================================================================" -ForegroundColor Cyan
    Write-Host (" Hardware : {0} | {1} GB RAM | {2} ({3} MB VRAM)" -f $Hw.CpuName, $Hw.RamTotalGB, $Hw.GpuName, $Hw.VramMB) -ForegroundColor Gray
    Write-Host (" Desemp.  : Slot $PerfSlot | PL {0}% | Undervolt + Leve OC (+{1} Core / +{2} Mem MHz)" -f $Hw.PerfPowerPct, $Plan.PerfCoreMHz, $Plan.PerfMemMHz) -ForegroundColor Gray
    Write-Host (" Economia : Slot $EcoSlot | PL {0}% | Undervolt de Eficiencia (+{1} Core base MHz)" -f $Hw.EcoPowerPct, $Plan.EcoBoostMHz) -ForegroundColor Gray
    Write-Host "----------------------------------------------------------------------------"
    Write-Host " [1] Calibrar Perfis Afterburner (UV + OC) + Ativar MODO DESEMPENHO agora"
    Write-Host " [2] Calibrar Perfis Afterburner (UV)      + Ativar MODO ECONOMIA agora"
    Write-Host " [3] Iniciar Monitor Automatico nesta janela (detecta jogos ao abrir/fechar)"
    Write-Host " [4] Instalar Automacao no Windows (roda oculto ao ligar o PC)"
    Write-Host " [5] Desinstalar Automacao e restaurar backups do MSI Afterburner"
    Write-Host " [6] Exibir Diagnostico Completo das Pecas e Status"
    Write-Host " [7] Corrigir Contraste / Filtro Branco (RGB Full 0-255, Gama 1.0 e DDC)"
    Write-Host " [0] Sair"
    Write-Host ""
    $op = Read-Host "Escolha uma opcao"

    switch ($op) {
        "1" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Apply-PerformanceMode -Hw $Hw -Schemes $Schemes -Interactive
            Pause
        }
        "2" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Apply-EconomyMode -Hw $Hw -Schemes $Schemes
            Pause
        }
        "3" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Write-Host "`nMonitor ativo! Pressione Ctrl+C para encerrar ou feche esta janela.`n" -ForegroundColor Green
            Start-GameWatcher
        }
        "4" {
            Ensure-AfterburnerProfiles -Hw $Hw -ForceOverwrite | Out-Null
            Install-AutoStartTask
            Pause
        }
        "5" {
            Uninstall-AutoStartTask
            Pause
        }
        "6" {
            Show-Diagnostics
            Pause
        }
        "7" {
            Ensure-DisplayContrastAndColor
            Write-Log "Verificacao de contraste, gama linear 1.0, RGB Full 0-255 e DDC concluida." "Green"
            Pause
        }
        "0" { exit 0 }
    }
}
```