name: NQ bias
on:
  schedule:
    - cron: "0 12 * * 1-5"
    - cron: "0 13 * * 1-5"
  workflow_dispatch:

permissions:
  contents: write

jobs:
  gate:
    runs-on: ubuntu-latest
    outputs:
      go: ${{ steps.t.outputs.go }}
    steps:
      - id: t
        run: |
          H=$(TZ=America/New_York date +%H)
          if [ "$H" = "08" ] || [ "${{ github.event_name }}" = "workflow_dispatch" ]; then
            echo "go=true" >> "$GITHUB_OUTPUT"
          fi

  run:
    needs: gate
    if: needs.gate.outputs.go == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install pandas yfinance
      - run: python nq_bias.py
      - run: |
          git config user.name "github-actions"
          git config user.email "actions@users.noreply.github.com"
          git add data.json
          git commit -m "NQ bias update" || exit 0
          git push
