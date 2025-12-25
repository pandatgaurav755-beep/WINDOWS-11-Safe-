name: RDP + Tailscale + (Optional) Chrome Remote Desktop (X)

on:
  workflow_dispatch:
    inputs:
      quick_test:         { description: "Run 5-minute test", type: boolean, default: false }
      runtime_minutes:    { description: "Runtime (max 360; default 355 when not test)", required: false, default: "355" }
      do_purge:           { description: "Purge bullet* devices at start (single instance only)", required: false, default: "true" }
      cycles:             { description: "0=stop after X; N=handoffs left incl this run", required: false, default: "0" }
      rdp_count:          { description: "How many RDP instances (1-10) [ignored if CRD provided]", required: false, default: "1" }
      crd_code:
        description: >-
          Paste raw CRD code (4/0AV...), full PowerShell command, or text/URL containing --code="4/...".
          If provided -> single instance; CRD launches first.
        required: false
        default: ""
      crd_pin:
        description: "CRD PIN (6–12 digits). Used when launching CRD to avoid interactive prompt."
        required: false
        default: "123456"

permissions:
  contents: read
  actions: write

defaults:
  run:
    shell: pwsh

jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.mk.outputs.matrix }}
      multi:  ${{ steps.mk.outputs.multi }}
    steps:
      - id: mk
        run: |
          $crd = @'
          ${{ inputs.crd_code }}
          '@.Trim()
          if ($crd.Length -gt 0) { $n = 1 } else {
            $n = [int]"${{ inputs.rdp_count }}"; if ($n -lt 1) { $n = 1 }; if ($n -gt 10) { $n = 10 }
          }
          $inc = @(); for ($i=1; $i -le $n; $i++){ $inc += @{ id = $i } }
          $json = @{ include = $inc } | ConvertTo-Json -Compress
          "matrix=$json" | Out-File $env:GITHUB_OUTPUT -Append -Encoding utf8
          $multi = if ($n -gt 1) { '1' } else { '0' }
          "multi=$multi" | Out-File -Append $env:GITHUB_OUTPUT -Encoding utf8
