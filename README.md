# demo-video

A Claude Code plugin that records a **narrated, captioned demo video of a pull request actually working** and attaches it to the PR.

- Real screen recordings, not screenshots: Chromium via [agent-browser](https://github.com/vercel-labs/agent-browser) (`web`) or Safari on the iOS Simulator via [agent-device](https://www.npmjs.com/package/agent-device) + `simctl` (`ios`, real WebKit)
- One narration sentence per scene, spoken with macOS `say` and burned in as a caption (reviewers are often muted)
- Deterministic assembly with ffmpeg: each scene lasts `max(video, audio)`, title cards between scenes, stream specs asserted before concatenation
- Verification Claude can actually do: a 3×3 contact sheet per scene, since it cannot play video
- Optional one-line attachment with `gh pr comment --attach` (gh ≥ 2.99, 2026-09). Without a PR it just produces the mp4 and prints the command

Everything runs locally. Narration text never leaves the machine.

Lineage: [Damien Tanner's demo-video gist](https://gist.github.com/dctanner/ee7fe7997bba0efbca49b8c0bdc1936a) (screenshots → cloud TTS → ffmpeg → R2), rebuilt around real recordings and local tools.

## Install

```bash
# try it without installing
claude --plugin-dir /path/to/demo-video

# or add this repo as a marketplace
/plugin marketplace add shiruten/demo-video
/plugin install demo-video@shiruten
```

Requirements (macOS): `ffmpeg`, `gh` 2.99+ (attach only), Xcode Command Line Tools (`swiftc`, for captions). Plus `npm i -g agent-browser` for `web`, or `npm i -g agent-device` and a booted iOS Simulator for `ios`.

## Use

```text
/demo-video:record <web|ios> [video-name] [#PR or PR URL] [scene requests...]

/demo-video:record web                       # record Chromium, print the attach command
/demo-video:record ios my-feature #123       # record Simulator Safari, then attach to PR #123 after confirmation
```

The skill only runs when you invoke it (`disable-model-invocation`). It writes `scenes.tsv`, records one file per scene, runs `scripts/build.sh`, shows you the result, and cleans up. Output lives in `~/Movies/pr-demo/<name>/` (`PR_DEMO_DIR` to change) and is deleted once the attachment is confirmed on the PR.

### Project notes

Put repo-specific facts in `.claude/demo-video.md` and the skill reads them at run time: test accounts, URLs, user roles, fixture recipes, brand colour, known pitfalls. The plugin itself stays generic.

### Narration voice

`build.sh` uses the system default voice unless you pass `--voice` (validated; `say` would otherwise fall back silently). For Japanese, `--voice Kyoko`. Captions use the system UI font, which falls back per script; override with `--font`.

## Layout

```text
skills/record/
├── SKILL.md                 workflow and decisions
├── references/
│   ├── ios-safari.md        pre-flight checks, recording, iOS input recipe
│   └── web-chromium.md      agent-browser setup and recording
└── scripts/
    ├── build.sh             scenes.tsv → say → captions → title cards → ffmpeg → verification → mp4 + summary.md
    ├── rec.sh               simctl recording start/stop (pid file)
    └── caption.swift        CoreText caption renderer (Homebrew ffmpeg has no drawtext)
```

---

## 日本語

PR の機能が**実際に動く様子を録画し、日本語ナレーションと字幕を付けた 1 本の mp4** にして PR に添付する Claude Code plugin です。

- 静止画ではなく実操作の録画。`web` は agent-browser（Chromium）、`ios` は agent-device + `simctl`（シミュレータの Safari、本物の WebKit）
- シーンごとに 1 文のナレーションを macOS の `say` で読み上げ、同じ文を字幕として焼き込み（GitHub ではミュートで見られることが多い）
- ffmpeg で決定的に組み立て。各シーンの長さは `max(映像, 音声)`、シーン間にタイトルカード、concat 前に全区間の諸元一致を検査
- Claude は動画を再生できないので、シーンごとの 3×3 コンタクトシートで検証
- PR 番号を渡せば `gh pr comment --attach` で添付（gh 2.99 以上）。渡さなければ mp4 を作ってコマンドを表示するだけ

すべてローカルで完結し、ナレーション文は外部に送られません。

### 使い方

```text
/demo-video:record <web|ios> [動画名] [#PR か PR の URL] [シーンの要望...]
```

ナレーションが日本語なら `build.sh` に `--voice Kyoko` を渡します（SKILL.md がそう指示します）。プロジェクト固有の事情（テストアカウント・URL・役割・フィクスチャ・ブランド色）は `.claude/demo-video.md` に書いておくと、実行時に読み込まれます。

### 必要なもの

macOS、`ffmpeg`、`gh` 2.99 以上（添付するときだけ）、Xcode Command Line Tools（字幕の描画）。`web` は `npm i -g agent-browser`、`ios` は `npm i -g agent-device` と起動済みの iOS シミュレータ。

## License

MIT
