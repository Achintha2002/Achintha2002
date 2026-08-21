name: Generate Trophy Image

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch: {}

jobs:
  generate-trophy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Generate trophy SVG
        env:
          GH_USERNAME: Achintha2002
          GH_TOKEN: ${{ secrets.TROPHY_TOKEN }}
          OUTPUT_PATH: trophy.svg
        run: |
          python3 << 'EOF'
          import os, sys, json, urllib.request

          USERNAME = os.environ["GH_USERNAME"]
          TOKEN = os.environ["GH_TOKEN"]
          OUTPUT_PATH = os.environ.get("OUTPUT_PATH", "trophy.svg")

          HEADERS = {
              "Authorization": f"token {TOKEN}",
              "Accept": "application/vnd.github+json",
              "User-Agent": USERNAME,
          }

          def get_json(url):
              req = urllib.request.Request(url, headers=HEADERS)
              with urllib.request.urlopen(req) as resp:
                  return json.loads(resp.read().decode())

          user = get_json(f"https://api.github.com/users/{USERNAME}")
          followers = user.get("followers", 0)
          public_repos = user.get("public_repos", 0)

          repos = get_json(f"https://api.github.com/users/{USERNAME}/repos?per_page=100")
          total_stars = sum(r.get("stargazers_count", 0) for r in repos)
          total_forks = sum(r.get("forks_count", 0) for r in repos)

          print(f"Followers: {followers}, Repos: {public_repos}, Stars: {total_stars}, Forks: {total_forks}")

          def rank_for(value, thresholds):
              for label, threshold in zip(["S", "A", "B", "C"], thresholds):
                  if value >= threshold:
                      return label
              return "C"

          trophies = [
              {"title": "Stars", "value": total_stars, "rank": rank_for(total_stars, [50, 20, 5])},
              {"title": "Followers", "value": followers, "rank": rank_for(followers, [50, 20, 5])},
              {"title": "Repositories", "value": public_repos, "rank": rank_for(public_repos, [30, 15, 5])},
              {"title": "Forks", "value": total_forks, "rank": rank_for(total_forks, [20, 10, 3])},
          ]

          RANK_COLORS = {"S": "#a855f7", "A": "#3b82f6", "B": "#22c55e", "C": "#a1a1aa"}
          CARD_W, CARD_H, GAP = 150, 150, 15
          COLS = len(trophies)
          TOTAL_W = CARD_W * COLS + GAP * (COLS - 1)

          svg_parts = [
              f'<svg width="{TOTAL_W}" height="{CARD_H}" viewBox="0 0 {TOTAL_W} {CARD_H}" xmlns="http://www.w3.org/2000/svg">',
              '<style>text{font-family:Segoe UI,Verdana,sans-serif;} .title{font-size:14px;fill:#c9d1d9;} .value{font-size:26px;font-weight:bold;} .rank{font-size:12px;fill:#8b949e;}</style>',
          ]

          for i, t in enumerate(trophies):
              x = i * (CARD_W + GAP)
              color = RANK_COLORS[t["rank"]]
              svg_parts.append(f'''
          <g transform="translate({x},0)">
            <rect x="0" y="0" width="{CARD_W}" height="{CARD_H}" rx="10" fill="#161b22" stroke="{color}" stroke-width="2"/>
            <text x="{CARD_W/2}" y="35" text-anchor="middle" class="title">{t["title"]}</text>
            <text x="{CARD_W/2}" y="80" text-anchor="middle" class="value" fill="{color}">{t["value"]}</text>
            <text x="{CARD_W/2}" y="115" text-anchor="middle" class="rank">Rank {t["rank"]}</text>
          </g>''')

          svg_parts.append("</svg>")

          with open(OUTPUT_PATH, "w") as f:
              f.write("\n".join(svg_parts))

          print(f"Trophy SVG written to {OUTPUT_PATH}")
          EOF

      - name: Commit and push trophy.svg
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add trophy.svg
          git diff --quiet && git diff --staged --quiet || git commit -m "Update trophy.svg"
          git push
