# Modo_Jogo_Automatico.bat — Perfis MSI Afterburner e Energia por Hardware (Criado por Antigravity)

> **Nota de Autoria:** Este script `.bat` universal foi criado por **Antigravity** (assistente de IA do Google DeepMind) com foco em automação segura baseada no hardware real de cada máquina, sem depender de configurações fixas de um computador específico.

## O que este script faz

O `Modo_Jogo_Automatico.bat` é um utilitário universal (Batch + PowerShell nativo do Windows 10/11) que **inspeciona as peças atuais do PC** (CPU, memória RAM livre/total, placa de vídeo dedicada/integrada, VRAM, temperatura, notebook vs. desktop, discos e planos de energia) para:

1. **Criar automaticamente dois perfis calibrados no MSI Afterburner** (com backup automático `.bak_modojogo` antes de alterar qualquer `.cfg`):
   - **Slot 4 (`[Profile4]` — Modo Desempenho):** libera o `PowerLimit` máximo seguro suportado pela placa (via leitura em tempo real do `nvidia-smi` quando disponível em GPUs NVIDIA, ou padrão seguro 100%), ajusta limite térmico seguro (`83 °C` em desktops / `80 °C` em notebooks) e mantém curva automática de ventoinhas.
   - **Slot 5 (`[Profile5]` — Modo Economia):** reduz o `PowerLimit` ao mínimo estável detectado para a GPU (entre `65%` e `85%`, com proteção térmica conservadora de `75 °C` e rotação silenciosa ao fechar jogos).
2. **Alternar o Plano de Energia do Windows e processos em segundo plano:**
   - **Ao abrir jogos pesados (Modo Desempenho):** ativa **Desempenho Máximo** (ou **Alto Desempenho**, criando o esquema caso não exista), aplica o **Perfil 4** do MSI Afterburner e reduz para prioridade `Idle` processos que disputam CPU/RAM/VRAM (como navegadores, indexadores e utilitários secundários). Em sistemas com pouca VRAM (`<= 4200 MB`) ou pouca RAM livre, oferece fechamento opcional de navegadores ou encerra processos auxiliares ociosos.
   - **Ao fechar todos os jogos pesados (Modo Economia):** aguarda `12` segundos de tolerância para evitar trocas durante alt-tab ou reinício rápido de partida, volta o Windows para **Economia de Energia**, aplica o **Perfil 5** do MSI Afterburner e restaura a prioridade normal dos processos.
3. **Automatização contínua em tempo real (CS2 e bibliotecas de jogos):**
   - Monitora automaticamente jogos pesados conhecidos (`cs2`, `valorant`, `fortniteclient-win64-shipping`, `r5apex`, `gta5`, `cyberpunk2077`, `eldenring`, `rdr2`, `cod`, etc.) **e** escaneia dinamicamente bibliotecas locais em todas as unidades (`SteamLibrary`, `Steam`, `Epic Games`, `Riot Games`, `XboxGames`, `GOG Games`, `EA Games`, `Games`, `Jogos`), ignorando instaladores, crash handlers e launchers.
   - Permite instalar uma **Tarefa Agendada oculta no Windows (`ModoJogoAutomatico_MSIAfterburner`)** que inicia silenciosamente ao fazer logon, sem janelas abertas na barra de tarefas.

---

## Como usar em qualquer PC

1. Baixe o arquivo [`Modo_Jogo_Automatico.bat`](./Modo_Jogo_Automatico.bat).
2. Clique com o botão direito e selecione **Executar como administrador** (ou dê duplo clique e aceite o UAC).
3. No menu interativo, escolha a opção desejada:
   - **`[1]` Calibrar Perfis + Ativar MODO DESEMPENHO agora:** analisa o hardware, grava os Slots `4` e `5` no MSI Afterburner e coloca o PC em desempenho máximo na hora.
   - **`[2]` Ativar MODO ECONOMIA agora:** alterna imediatamente para o perfil econômico (GPU + Windows).
   - **`[3]` Iniciar Monitor Automático nesta janela:** acompanha os jogos em tempo real exibindo o log na tela (`Ctrl+C` para encerrar).
   - **`[4]` Instalar Automação no Windows (segundo plano):** registra o monitor automático para rodar oculto sempre que o Windows iniciar.
   - **`[5]` Desinstalar Automação e restaurar backups:** remove a tarefa agendada e restaura os arquivos originais `.bak_modojogo` do MSI Afterburner.
   - **`[6]` Diagnóstico Completo:** mostra CPU, RAM, GPU, VRAM, calibração calculada, GUIDs de energia, discos e total de jogos detectados.

### Parâmetros de linha de comando (opcional)
- `Modo_Jogo_Automatico.bat -desempenho` — aplica Modo Desempenho diretamente e sai.
- `Modo_Jogo_Automatico.bat -economia` — aplica Modo Economia diretamente e sai.
- `Modo_Jogo_Automatico.bat -monitor` — inicia o monitor automático com janela visível.
- `Modo_Jogo_Automatico.bat -auto` — inicia o monitor automático em modo silencioso (`%TEMP%\ModoJogoAutomatico.log`).

---

## Código Interno do `Modo_Jogo_Automatico.bat`

```bat
@echo off
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
$PerfCoreOffsetMHz = 0       # Ex: 50 para +50 MHz (0 = clock Boost padrao seguro da placa)
$PerfMemOffsetMHz  = 0       # Ex: 200 para +200 MHz (0 = memoria padrao segura)
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

    $markerFile = Join-Path $ab.ProfilesDir ".modojogo_profiles_v1"
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

    $perfCoreKHz = $PerfCoreOffsetMHz * 1000
    $perfMemKHz  = $PerfMemOffsetMHz * 1000
    $perfTherm   = if ($Hw.IsLaptop) { 80 } else { 83 }
    $ecoFanMode  = if ($Hw.IsLaptop) { 0 } else { 1 }
    $ecoFanPct   = if ($Hw.IsLaptop) { 35 } else { 30 }

    $perfLines = @(
        "Format=2",
        "PowerLimit=$($Hw.PerfPowerPct)",
        "ThermalLimit=$perfTherm",
        "ThermalPrioritize=0",
        "CoreClkBoost=$perfCoreKHz",
        "MemClkBoost=$perfMemKHz",
        "FanMode=0",
        "FanSpeed=60"
    )
    $ecoLines = @(
        "Format=2",
        "PowerLimit=$($Hw.EcoPowerPct)",
        "ThermalLimit=75",
        "ThermalPrioritize=0",
        "CoreClkBoost=0",
        "MemClkBoost=0",
        "FanMode=$ecoFanMode",
        "FanSpeed=$ecoFanPct"
    )

    foreach ($cfg in $gpuCfgs) {
        $bak = "$($cfg.FullName).bak_modojogo"
        if (-not (Test-Path $bak)) { Copy-Item $cfg.FullName $bak -Force }
        Set-IniSection -FilePath $cfg.FullName -SectionName "Profile$PerfSlot" -KeyValueLines $perfLines
        Set-IniSection -FilePath $cfg.FullName -SectionName "Profile$EcoSlot"  -KeyValueLines $ecoLines
        Write-Log ("Perfis calibrados gravados em $($cfg.Name): Slot $PerfSlot [Desempenho PL=$($Hw.PerfPowerPct)%] | Slot $EcoSlot [Economia PL=$($Hw.EcoPowerPct)%]") "Green"
    }

    foreach ($slot in @($PerfSlot, $EcoSlot)) {
        $slotFile = Join-Path $ab.ProfilesDir "Profile$slot.cfg"
        if (-not (Test-Path $slotFile)) {
            Set-Content -Path $slotFile -Value "[Settings]`r`nProfileContents=1" -Encoding Default -NoNewline
        }
    }

    Set-Content -Path $markerFile -Value ("Calibrado em " + (Get-Date -Format s) + " para " + $Hw.GpuName) -Encoding UTF8

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
    Set-BackgroundAppsThrottled -Throttle $true -Hw $Hw -InteractiveClosePrompt:$Interactive
    Write-Log "MODO DESEMPENHO ATIVADO -> Plano de Energia Maximo + Afterburner Perfil $PerfSlot (PL $($Hw.PerfPowerPct)%)." "Green" -Silent:$Silent
}

function Apply-EconomyMode {
    param($Hw, $Schemes, [switch]$Silent)
    if ($Schemes.EcoGuid) {
        powercfg /setactive $Schemes.EcoGuid | Out-Null
    }
    Ensure-AfterburnerProfiles -Hw $Hw | Out-Null
    Invoke-AfterburnerProfile -Slot $EcoSlot
    Set-BackgroundAppsThrottled -Throttle $false -Hw $Hw
    Write-Log "MODO ECONOMIA ATIVADO -> Plano Economia + Afterburner Perfil $EcoSlot (PL $($Hw.EcoPowerPct)%)." "Yellow" -Silent:$Silent
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
        $markerFile = Join-Path $ab.ProfilesDir ".modojogo_profiles_v1"
        if (Test-Path $markerFile) { Remove-Item $markerFile -Force }
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
    $games   = Get-DiscoveredGameExes
    $task    = Get-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue

    Write-Host "`n=== DIAGNOSTICO DO HARDWARE ATUAL ===" -ForegroundColor Cyan
    Write-Host (" CPU      : {0} ({1} nucleos / {2} threads)" -f $Hw.CpuName, $Hw.CpuCores, $Hw.CpuThreads)
    Write-Host (" RAM      : {0} GB livres / {1} GB totais" -f $Hw.RamFreeGB, $Hw.RamTotalGB)
    Write-Host (" Tipo PC  : {0}" -f $(if ($Hw.IsLaptop) { "Notebook (Bateria)" } else { "Desktop" }))
    Write-Host (" GPU      : {0} ({1} | {2} MB VRAM{3})" -f $Hw.GpuName, $Hw.GpuVendor, $Hw.VramMB, $(if ($Hw.GpuTempC) { " | $($Hw.GpuTempC) C" } else { "" }))
    Write-Host (" Calibr.  : Afterburner Desempenho (Slot $PerfSlot) = PL {0}% | Economia (Slot $EcoSlot) = PL {1}%" -f $Hw.PerfPowerPct, $Hw.EcoPowerPct)
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
}

$Hw      = Get-HardwareProfile
$Schemes = Get-PowerSchemeIds

while ($true) {
    Clear-Host
    Write-Host "============================================================================" -ForegroundColor Cyan
    Write-Host "     MODO JOGO AUTOMATICO (HARDWARE-AWARE + PERFIS MSI AFTERBURNER)" -ForegroundColor Cyan
    Write-Host "============================================================================" -ForegroundColor Cyan
    Write-Host (" Hardware : {0} | {1} GB RAM | {2} ({3} MB VRAM)" -f $Hw.CpuName, $Hw.RamTotalGB, $Hw.GpuName, $Hw.VramMB) -ForegroundColor Gray
    Write-Host (" Perfis   : Slot $PerfSlot = Desempenho (PL {0}%) | Slot $EcoSlot = Economia (PL {1}%)" -f $Hw.PerfPowerPct, $Hw.EcoPowerPct) -ForegroundColor Gray
    Write-Host "----------------------------------------------------------------------------"
    Write-Host " [1] Calibrar Perfis Afterburner pelo Hardware + Ativar MODO DESEMPENHO agora"
    Write-Host " [2] Ativar MODO ECONOMIA agora (apos jogar / uso leve)"
    Write-Host " [3] Iniciar Monitor Automatico nesta janela (detecta jogos ao abrir/fechar)"
    Write-Host " [4] Instalar Automacao no Windows (roda oculto ao ligar o PC)"
    Write-Host " [5] Desinstalar Automacao e restaurar backups do MSI Afterburner"
    Write-Host " [6] Exibir Diagnostico Completo das Pecas e Status"
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
        "0" { exit 0 }
    }
}
```