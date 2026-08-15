# OTX Pulse GitHub Action

Automate the creation and submission of threat intelligence pulses to AlienVault OTX directly from your GitHub Actions workflows. This action allows you to validate and parse inputs securely, passing indicators of compromise (IoCs), tags, references, and more to your OTX account seamlessly.

## Features

- **Automated Submission**: Quickly publish threat intelligence data (Pulses) as part of your CI/CD pipeline.
- **Strict Validation**: Securely handles string lists, arrays, and JSON indicators before submitting.
- **Extensive Metadata**: Supports adding adversary info, TLP levels, targeted countries, industries, malware families, and MITRE ATT&CK IDs.
- **Privacy Controls**: Easy toggles to make pulses public or keep them private to specific group IDs.

## Usage Example

Below is an example of how to use this action in a workflow that triggers on commit, preparing malicious domain indicators and submitting them to OTX:

```yaml
name: OTX Pulse Dispatch on Commit
on:
  push:
    branches: [ main ]

jobs:
  dispatch-pulse:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Prepare OTX indicators
        id: indicators
        run: |
          if [ -f nrd_suspicious_domains.txt ]; then
            INDICATORS=$(awk 'NF' nrd_suspicious_domains.txt | jq -R -s -c 'split("\n")[:-1] | map({indicator: ., type: "domain", title: "Suspicious NRD", description: "Potentially malicious newly-registered domain", role: "phishing_host"})')
            echo "indicators=$INDICATORS" >> $GITHUB_OUTPUT
          else
            echo "indicators=[]" >> $GITHUB_OUTPUT
          fi

      - name: Submit Pulse to OTX
        uses: julioliraup/otx-pulse@main
        with:
          otx-api-key: ${{ secrets.OTX_API_KEY }}
          name: "Suspicious NRD Domains Detected"
          description: "A set of newly-registered domains has been flagged as suspicious by the Antiphishing pipeline. These domains were analyzed for typosquatting and brand desensitization across 14 detection algorithms. There is a chance these are false-positives, please review before action."
          public: "false"
          tlp: "amber"
          tags: "suspicious,nrd,phishing,typosquatting"
          references: "https://github.com/julioliraup/Antiphishing"
          adversary: "Unknown Scanners"
          indicators: ${{ steps.indicators.outputs.indicators }}
```

## Inputs

| Name                 | Description                                                        | Required | Default |
|----------------------|--------------------------------------------------------------------|----------|---------|
| `otx-api-key`        | Your OTX API Key (X-OTX-API-KEY header)                            | Yes      |         |
| `name`               | Title of the Pulse                                                 | Yes      |         |
| `description`        | Detailed description of the threat                                 | No       | `""`    |
| `public`             | Make the pulse public (`true` or `false`)                          | No       | `false` |
| `tlp`                | Traffic Light Protocol level (white, green, amber, red)            | No       | `green` |
| `tags`               | Comma-separated list of tags                                       | No       | `""`    |
| `references`         | Comma-separated list of reference URLs                             | No       | `""`    |
| `adversary`          | Name of the threat actor or adversary group                        | No       | `""`    |
| `targeted-countries` | Comma-separated list of targeted countries                         | No       | `""`    |
| `industries`         | Comma-separated list of targeted industries                        | No       | `""`    |
| `group-ids`          | Comma-separated list of integer group IDs allowed to view          | No       | `""`    |
| `malware-families`   | Comma-separated list of malware families                           | No       | `""`    |
| `attack-ids`         | Comma-separated list of MITRE ATT&CK IDs (e.g., T1003)             | No       | `""`    |
| `indicators`         | Strict JSON array containing threat indicators                     | No       | `[]`    |

## Support

If you find this action useful in your security pipelines, consider supporting the continued development and maintenance of my open-source work:
[Support the Developer](https://github.com/sponsors/julioliraup)
