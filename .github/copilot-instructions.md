# Copilot Instructions for ClashRuleSet

## Project Overview

ClashRuleSet is a curated collection of domain/IP filtering rules for **Clash** (network proxy tool). It provides specialized traffic splitting for AI services, ByteDance apps, Web3, education, and developer tools, plus standard streaming/gaming services.

**Base**: Originally derived from ACL4SSR, customized for personal use.

## Architecture

### Directory Structure

- **`Clash/`** - Main ruleset repository
  - **Config Files**: `kclash.ini` (ruleset + proxy groups), `pref.ini` (generator settings)
  - **`.list` files** - Domain/IP filter rules (top-level and in `Ruleset/` subdirectory)
  - **`Ruleset/`** - Specialized service-specific rules (~100+ files)

### Core Concepts

**Rule Lists** (`.list` files)

- Format: Plain text with domain/IP patterns
- Pattern types: `DOMAIN-SUFFIX`, `DOMAIN-KEYWORD`, `DOMAIN`, `IP-CIDR`, `IP-CIDR6`, `GEOIP`
- Comments describe rule purpose/content sources (e.g., `# 内容：OpenAi,claude,recraft,poe,bing,gemini,perplexity`)
- Example: [Clash/Ruleset/AiDomain.list](Clash/Ruleset/AiDomain.list) (AI services including OpenAI, Claude, Gemini, Perplexity)

**Configuration Files** (INI format)

- `kclash.ini`: Connects rules to proxy groups
  - `[custom]` section: `ruleset=label`URL pairs + `custom_proxy_group` definitions
  - Proxy groups use backtick-separated fields: label, type, node filters and optional built-in policies; automatic groups also specify a test URL and interval
- `pref.ini`: Rule generator settings (exclude patterns, base configs, proxy settings)

## Key Patterns & Conventions

### Ruleset Organization

**Specialization**:

- AI-focused: `AiDomain.list` (US nodes), `CiciAi.list` (Singapore), `Perplexity.list`
- ByteDance: `ByteDance.list`
- Web3: `Web3.list`, `Crypto.list`
- Education: `Khan.list`, `Edutools.list`
- Standard: `Netflix.list`, `YouTube.list`, `Spotify.list`, `Bilibili.list`
- China-specific: `ChinaDomain.list` (direct), `ChinaCompanyIp.list`, `ChinaMedia.list`

**Naming**: Service name + optional suffix (e.g., `Netflix.list`, `NetflixIP.list`)

### Configuration Patterns

**Proxy Group Members** (in `kclash.ini` and `pref.ini`):

```
custom_proxy_group=🇺🇲 美区AI`select`[]♻️ AI自动切换`(?i)(美国|\bUS\b)`(?i)(日本|\bJP\b)
custom_proxy_group=♻️ AI自动切换`fallback`(?i)(美国|\bUS\b)`(?i)(日本|\bJP\b)`https://www.gstatic.com/generate_204`60
custom_proxy_group=🔒 Web3`select`.*`[]DIRECT
custom_proxy_group=🛑 广告拦截`select`[]REJECT`[]DIRECT
```

- `.*` includes actual nodes read from the subscription by the converter
- Explicit `[]` members are built-in policies such as DIRECT and REJECT, except for the AI automatic-switch option below
- Keep service groups; do not add standalone region groups, custom groups, or other automatic selection groups or group-to-group references
- Exception: 美区AI is a select group with AI自动切换 first (the default), followed by actual US and Japan nodes for manual selection
- AI自动切换 uses fallback with US nodes first, then Japan nodes, and a 60-second connectivity check; it contains actual nodes only
- Keep DIRECT/REJECT options and preserve whether the default is a built-in policy or an actual node
- INI conversion does not itself configure native Clash proxy-providers

**Region-to-Service Mapping**:

- US nodes → AI services, with Japan nodes as automatic fallback when all US nodes are unavailable
- Singapore → ByteDance/CiciAI
- Direct (DIRECT) → CN content + Edu
- Each service uses actual subscription nodes directly; 美区AI additionally offers AI自动切换 as its default, and 🚀 节点选择 is used by rules only

### File Size Convention

- Large lists (500+ lines): `Netflix.list`, `Developer.list`, `GoogleCNProxyIP.list`
- Medium (100-500 lines): Most service rulesets
- Small (<100 lines): Ad-blocking rules (`BanAD.list`)

## Developer Workflows

### Adding/Updating Rules

1. Identify service category and create/update `.list` file in appropriate location
2. Use standard pattern format: `DOMAIN-SUFFIX,example.com` (no spaces after comma)
3. Add comment header describing content source and services included
4. For new services: Add entry to `kclash.ini` ruleset section and create corresponding proxy_group

### Testing/Usage

- Both `kclash.ini` and `pref.ini` can be used via URL parameter (subscriber appends `&config=...`)
- No build process; rules are consumed directly by Clash or parameter processors
- Platform targets: Clash (iOS/macOS/Windows), iOS Shadowrocket, etc.

### Common Tasks

**Add new service ruleset**:

- Create `Clash/Ruleset/ServiceName.list`
- Add rule entry to `kclash.ini` in `[custom]` section
- Add proxy_group entry (or reuse existing group if appropriate)

**Categorize domains**:

- Research service's domain/IP ranges
- Add patterns to appropriate service ruleset
- Document any recommended node region; keep group members limited to actual subscription nodes and built-in policies

## External Dependencies & Data Sources

- **Primary**: Clash/Shadowrocket subscriptions (provide nodes)
- **Rule Sources**: Comments in `.list` files reference original rule sources (ACL4SSR, custom research)
- **GitHub**: Raw links to this repo for remote ruleset loading in `kclash.ini`
- **Generator**: External tools (SubConverter, etc.) consume `.ini` config to generate platform-specific configs

## Integration Points

- **Subscription providers**: Use `&config=https://raw.githubusercontent.com/.../kclash.ini` URL parameter
- **Clash clients**: Load config via URL or local file
- **Parameter processors**: Internal server parses `.ini` and generates platform configs (Surge, Surfboard, Mellow, etc.)

## Critical Conventions

- **No executable code** - This is pure configuration/data
- **CHN-to-DIRECT, GFW proxy-only**: [ChinaDomain.list](Clash/ChinaDomain.list) exempts CN domains; [ProxyGFWlist.list](Clash/ProxyGFWlist.list) proxies blocked sites
- **Order matters in kclash.ini**: Earlier rules take precedence; place specific rules before general ones
- **Comment formatting**: Use `# content description` format at top of `.list` files for clarity
- **Region node naming**: Uses emoji + Chinese+English (🇺🇲 美区AI = US AI region)
