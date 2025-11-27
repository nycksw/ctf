# CTF Write-ups

Personal notes from HackTheBox machines and PortSwigger Web Security Academy labs.

## Selected Write-ups

### Active Directory

**[EscapeTwo](md/htb-escapetwo-20250116.md)** - SMB enumeration, MSSQL `xp_cmdshell`, and certificate forgery via misconfigured CA template.

**[Flight](md/htb-flight-20241008.md)** - LFI to NTLM hash capture, SMB lateral movement, GodPotato to DCSync.

**[Sauna](md/htb-sauna-20240831.md)** - Kerbrute, ASREP-roasting, AutoLogon credentials from registry.

**[Search](md/htb-search-20240920.md)** - BloodHound analysis and credential chaining.

### Web Applications

**[MonitorsThree](md/htb-monitorsthree-20241203.md)** - SQLi, XML signature bypass for Cacti RCE, Duplicati privesc.

**[Clicker](md/htb-clicker-20241004.md)** - Source code from NFS share, SQLi for role elevation, `PERL5OPT` abuse.

**[SolarLab](md/htb-solarlab-20240605.md)** - Multiple RCE vectors, credential decryption.

**[iClean](md/htb-iclean-20240529.md)** - SSTI and XSS.

### Source Code Analysis

**[UpDown](md/htb-updown-20241216.md)** - Git extraction, upload bypass, Python 2.7 injection, `easy_install` sudo.

**[Bizness](md/htb-bizness-20240108.md)** - OFBiz RCE, Derby database hash extraction.

### Cookie Security (PortSwigger)

**[SameSite Strict Bypass via Client-side Redirect](md/psa-samesite-strict-bypass-via-client-side-redirect-20250604.md)** - DOM redirect to maintain same-site context.

**[SameSite Lax Bypass via Cookie Refresh](md/psa-samesite-lax-bypass-via-cookie-refresh-20250604.md)** - Two-minute window after cookie refresh.

---

## Index

### 2025

| Date | Write-up |
|------|----------|
| 2025-08-06 | [HTB: Wifinetic](md/htb-wifinetic-20250806.md) |
| 2025-08-06 | [HTB: Legacy](md/htb-legacy-20250806.md) |
| 2025-08-05 | [HTB: Outbound](md/htb-outbound-20250805.md) |
| 2025-07-31 | [HTB: Nocturnal](md/htb-nocturnal-20250731.md) |
| 2025-06-14 | [PSA: CORS Basic Origin Reflection](md/psa-cors-basic-origin-reflection-20250614.md) |
| 2025-06-13 | [PSA: SQLi Union Attack - Finding Number of Columns](md/psa-sqli-union-attack-finding-number-of-columns-20250613.md) |
| 2025-06-13 | [PSA: SQLi Union Attack - Finding a Column Containing Text](md/psa-sqli-union-attack-finding-a-column-containing-text-20250613.md) |
| 2025-06-13 | [PSA: SQLi - Querying Database Type and Version on Oracle](md/psa-sqli-querying-the-database-type-and-version-on-oracle-20250613.md) |
| 2025-06-11 | [PSA: DOM XSS in AngularJS Expression](md/psa-dom-xss-in-angularjs-expression-with-angle-brackets-and-double-quotes-html-encoded-20250611.md) |
| 2025-06-11 | [PSA: DOM XSS in document.write Inside Select Element](md/psa-dom-xss-in-document.write-sink-using-source-location.search-inside-a-select-element-20250611.md) |
| 2025-06-11 | [PSA: DOM XSS in innerHTML Sink](md/psa-dom-xss-in-innerhtml-sink-using-source-location.search-20250611.md) |
| 2025-06-11 | [PSA: DOM XSS in jQuery Selector via Hashchange](md/psa-dom-xss-in-jquery-selector-sink-using-a-hashchange-event-20250611.md) |
| 2025-06-11 | [PSA: DOM XSS in jQuery Anchor href Attribute](md/psa-dom-xss-in-jquery-anchor-href-attribute-sink-using-location.search-source-20250611.md) |
| 2025-06-10 | [PSA: DOM XSS in document.write Sink](md/psa-dom-xss-in-document.write-sink-using-source-location.search-20250610.md) |
| 2025-06-04 | [PSA: SameSite Strict Bypass via Client-side Redirect](md/psa-samesite-strict-bypass-via-client-side-redirect-20250604.md) |
| 2025-06-04 | [PSA: SameSite Lax Bypass via Cookie Refresh](md/psa-samesite-lax-bypass-via-cookie-refresh-20250604.md) |
| 2025-06-03 | [PSA: SameSite Lax Bypass](md/psa-samesite-lax-bypass-20250603.md) |
| 2025-06-03 | [PSA: XSS to CSRF](md/psa-xss-csrf-20250603.md) |
| 2025-06-03 | [PSA: XSS to CSRF - Capturing Passwords](md/psa-xss-csrf-capturing-passwords-20250603.md) |
| 2025-06-03 | [PSA: CSRF Token Tied to Non-session Cookie](md/psa-csrf-token-tied-to-non-session-cookie-20250603.md) |
| 2025-06-03 | [PSA: CSRF Token Validation Depends on Request Method](md/psa-csrf-where-token-validation-depends-on-request-method-20250603.md) |
| 2025-04-18 | [HTB: Cap](md/htb-cap-20250418.md) |
| 2025-04-17 | [HTB: Secret](md/htb-secret-20250417.md) |
| 2025-01-16 | [HTB: EscapeTwo](md/htb-escapetwo-20250116.md) |

### 2024

| Date | Write-up |
|------|----------|
| 2024-12-18 | [HTB: Mentor](md/htb-mentor-20241218.md) |
| 2024-12-16 | [HTB: UpDown](md/htb-updown-20241216.md) |
| 2024-12-03 | [HTB: MonitorsThree](md/htb-monitorsthree-20241203.md) |
| 2024-12-03 | [HTB: Alert](md/htb-alert-20241203.md) |
| 2024-11-22 | [HTB: Sea](md/htb-sea-20241122.md) |
| 2024-11-13 | [HTB: Instant](md/htb-instant-20241113.md) |
| 2024-10-08 | [HTB: Flight](md/htb-flight-20241008.md) |
| 2024-10-04 | [HTB: Clicker](md/htb-clicker-20241004.md) |
| 2024-10-04 | [HTB: Escape](md/htb-escape-20241004.md) |
| 2024-10-01 | [HTB: Forest](md/htb-forest-20241001.md) |
| 2024-09-27 | [HTB: Broker](md/htb-broker-20240927.md) |
| 2024-09-27 | [HTB: Support](md/htb-support-20240927.md) |
| 2024-09-26 | [HTB: Editorial](md/htb-editorial-20240926.md) |
| 2024-09-22 | [HTB: ServMon](md/htb-servmon-20240922.md) |
| 2024-09-20 | [HTB: Caption](md/htb-caption-20240920.md) |
| 2024-09-20 | [HTB: Search](md/htb-search-20240920.md) |
| 2024-09-13 | [HTB: Sightless](md/htb-sightless-20240913.md) |
| 2024-09-10 | [HTB: Cascade](md/htb-cascade-20240910.md) |
| 2024-09-09 | [HTB: Jarvis](md/htb-jarvis-20240909.md) |
| 2024-09-09 | [HTB: Netmon](md/htb-netmon-20240909.md) |
| 2024-09-09 | [HTB: Remote](md/htb-remote-20240909.md) |
| 2024-09-04 | [HTB: Jerry](md/htb-jerry-20240904.md) |
| 2024-09-04 | [HTB: Love](md/htb-love-20240904.md) |
| 2024-08-31 | [HTB: Active](md/htb-active-20240831.md) |
| 2024-08-31 | [HTB: Arctic](md/htb-arctic-20240831.md) |
| 2024-08-31 | [HTB: Bounty](md/htb-bounty-20240831.md) |
| 2024-08-31 | [HTB: Buff](md/htb-buff-20240831.md) |
| 2024-08-31 | [HTB: Sauna](md/htb-sauna-20240831.md) |
| 2024-08-09 | [HTB: Tabby](md/htb-tabby-20240809.md) |
| 2024-08-07 | [HTB: Sunday](md/htb-sunday-20240807.md) |
| 2024-08-07 | [HTB: SwagShop](md/htb-swagshop-20240807.md) |
| 2024-08-06 | [HTB: Precious](md/htb-precious-20240806.md) |
| 2024-07-01 | [HTB: Networked](md/htb-networked-20240701.md) |
| 2024-07-01 | [HTB: OpenAdmin](md/htb-openadmin-20240701.md) |
| 2024-06-28 | [HTB: Bashed](md/htb-bashed-20240628.md) |
| 2024-06-28 | [HTB: Irked](md/htb-irked-20240628.md) |
| 2024-06-24 | [HTB: Blurry](md/htb-blurry-20240624.md) |
| 2024-06-07 | [HTB: Pandora](md/htb-pandora-20240607.md) |
| 2024-06-05 | [HTB: SolarLab](md/htb-solarlab-20240605.md) |
| 2024-06-04 | [HTB: WifineticTwo](md/htb-wifinetictwo-20240604.md) |
| 2024-05-30 | [HTB: Mailing](md/htb-mailing-20240530.md) |
| 2024-05-30 | [HTB: Ranking and Points](md/htb-ranking-and-points-20240530.md) |
| 2024-05-29 | [HTB: iClean](md/htb-iclean-20240529.md) |
| 2024-05-29 | [HTB: Perfection](md/htb-perfection-20240529.md) |
| 2024-05-28 | [HTB: Headless](md/htb-headless-20240528.md) |
| 2024-05-27 | [HTB: BoardLight](md/htb-boardlight-20240527.md) |
| 2024-05-27 | [HTB: Usage](md/htb-usage-20240527.md) |
| 2024-05-15 | [HTB: Busqueda](md/htb-busqueda-20240515.md) |
| 2024-05-13 | [HTB: Previse](md/htb-previse-20240513.md) |
| 2024-05-13 | [HTB: Surveillance](md/htb-surveillance-20240513.md) |
| 2024-01-08 | [HTB: Bizness](md/htb-bizness-20240108.md) |

### 2023

| Date | Write-up |
|------|----------|
| 2023-11-27 | [HTB: Devvortex](md/htb-devvortex-20231127.md) |
| 2023-11-18 | [HTB: Nibbles](md/htb-nibbles-20231118.md) |
| 2023-11-14 | [HTB: Pilgrimage](md/htb-pilgrimage-20231114.md) |
| 2023-11-14 | [HTB: Topology](md/htb-topology-20231114.md) |
| 2023-11-13 | [HTB: Armageddon](md/htb-armageddon-20231113.md) |
| 2023-11-13 | [HTB: Codify](md/htb-codify-20231113.md) |
| 2023-11-13 | [HTB: PC](md/htb-pc-20231113.md) |
| 2023-11-13 | [HTB: Sense](md/htb-sense-20231113.md) |
| 2023-10-23 | [HTB: Horizontall](md/htb-horizontall-20231023.md) |
| 2023-10-19 | [HTB: Drive](md/htb-drive-20231019.md) |
| 2023-10-19 | [HTB: ScriptKiddie](md/htb-scriptkiddie-20231019.md) |
| 2023-10-12 | [HTB: Paper](md/htb-paper-20231012.md) |
| 2023-10-10 | [HTB: Analytics](md/htb-analytics-20231010.md) |
| 2023-10-05 | [HTB: Shocker](md/htb-shocker-20231005.md) |
| 2023-10-04 | [HTB: Blocky](md/htb-blocky-20231004.md) |
| 2023-10-04 | [HTB: Mirai](md/htb-mirai-20231004.md) |
| 2023-10-03 | [HTB: Knife](md/htb-knife-20231003.md) |
| 2023-10-03 | [HTB: Sau](md/htb-sau-20231003.md) |
| 2023-10-02 | [HTB: CozyHosting](md/htb-cozyhosting-20231002.md) |
| 2023-10-02 | [HTB: Keeper](md/htb-keeper-20231002.md) |
