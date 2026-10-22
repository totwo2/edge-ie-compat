English | [简体中文](README.md)

# Edge IE Compatibility Manager (edge-ie-compat)

**One-liner:** Get your colleagues' legacy intranet sites to open correctly in Edge using the IE engine — just hand over the URLs, and the AI figures out which IE version each site needs, then exports a ready-to-share `.bat` your whole team can reuse.

## Who it's for

- Your company still runs legacy systems (OA, finance, industrial control) that only work on the IE engine.
- You want to add these sites to Edge's IE compatibility mode (Enterprise Mode / IE mode).
- After setting things up once, you need to send the same set of site configurations to colleagues to use directly.

## Three things it does for you

1. **Guided version probing.** Nobody can guess whether an old system needs the IE8, IE9, or IE11 engine — it has to be tested. The AI walks through IE9 → IE8 → IE11 in order on your behalf; you just watch whether the page works and reply "works / doesn't".
2. **Add / remove / edit / list sites.** Tell the AI in plain language to add a site, delete a site, change its document mode, or list everything; it gives you a one-line summary to keep.
3. **One-click `.bat` export.** Once tuned, run the generator once to produce `setup-ie-compat.bat`. Run it as administrator and it writes the sites into Edge's Enterprise Mode list + the Intranet zone + ActiveX policy; reopen Edge and it takes effect. Send this `.bat` to colleagues in your company and they just run it as administrator too.

## 30-second quick start

1. Copy `sitelist.xml.example` from the skill folder into `sitelist.xml`, and replace the placeholder fake domain `intranet.example.com:7001` with your real intranet site(s) — you can list several at once.
2. Run the generator in the skill folder:

```
cd ~/.workbuddy/skills/edge-ie-compat
python3 gen.py
```

3. On Windows, right-click the generated `dist/setup-ie-compat.bat` and choose "Run as administrator", then reopen Edge and open your legacy system to check it works.
4. Not sure which version to use? Just hand the URL to the AI and run the guided test below.

## Guided IE version test (required for old sites)

1. Give the AI the URL (with port) and say whether you roughly know which version you need.
2. By default the AI writes the first working one (IE9 → IE8 → IE11) into `sitelist.xml`, runs the generator, and hands you the `.bat` path.
3. On your side: save anything unsaved in Edge → run the `.bat` → reopen Edge → open the site and glance over the checklist below.
4. Reply "works / doesn't" to the AI:
   - Works → lock that mode; debugging done.
   - Doesn't → the AI switches to the next engine, regenerates, and you test again, until it passes or all three are tried.
5. Repeat this flow site by site; for sites sharing the same system / tech stack, test one representative site first and apply the result to the rest.

> ⚠️ After handing you the `.bat`, the AI stops and waits for your real-world feedback — it will not assume success and move on by itself. This is the key to avoiding a whole debugging session failing silently.

### What counts as "works" (try the most common functions: log in / query / submit)

| Counts as works | Counts as doesn't |
|---|---|
| Page opens, no white screen | White screen / endless spinner / "page can't be displayed" |
| Can log in normally | Prompts for missing plugin or control / "This site needs Internet Explorer" |
| Common buttons respond to clicks | No response on click / can't submit / data won't load |
| Tables and text render correctly, no layout break | Broken layout / text too big or too small / broken images |

### When all three modes fail (fallback)

- This tool only supports IE8 / IE9 / IE11 document modes; IE7 and older engines are out of scope. If you suspect an older engine, go through IT or the system vendor.
- Things to try: check for a missing Java plugin or old IE component, whether 32-bit IE is needed, add the site in IE's Compatibility View, or contact the vendor to confirm the tech stack.
- If it still says "missing plugin / control" and you've confirmed it's in IE mode, it may be an ActiveX that isn't marked safe for scripting (the generator disables this by default as a conservative security setting). That's a security trade-off only worth opening up for trusted intranet sites — when it happens, just ask the AI to regenerate with it enabled; you don't need to edit the `.bat` by hand.

## Managing sites (add / remove / edit / list)

All sites live in `sitelist.xml` (the single source of truth). Tell the AI in plain language:

- **Add:** give the URL + mode (or just run the guided test); the AI appends a line inside `<site-list>`, and `version` is auto-incremented by the generator — you don't need to touch it.
- **Remove:** the AI deletes the corresponding `<site>` line and reruns the generator.
- **Edit:** the AI changes that line's `<compat-mode>` and reruns the generator.
- **List:** the AI reads `sitelist.xml`, reads back the current sites + modes, and gives you a summary sentence like "what mode each site ended up using" so you can forward it to colleagues.

After any change, run `python3 gen.py` once so the version increments and the script refreshes.

## What the generator does

`gen.py` reads `sitelist.xml` → increments `version` by +1 → auto-derives the ZoneMap hosts from the list → generates `dist/setup-ie-compat.bat` (with the Edge Enterprise Mode policy + Intranet zone + ActiveX + restart, idempotent and re-runnable). The `.bat` self-elevates (UAC prompt on double-click); registry / zone entries use `/f` to overwrite, so re-running is safe.

Safety guards (bad input is rejected outright with an error — it never produces a broken script):

- Site URLs may not contain spaces, quotes, `;`, `\`, `<>`, `&`, or other special / non-ASCII characters (to avoid breaking the `.bat` or injecting commands, and to avoid GBK mojibake).
- Document mode only accepts IE8 / IE9 / IE11; open-in mode only accepts IE11.
- Empty list, missing version, or invalid XML all abort with a reason.

On success it prints: `version=N, sites=M, zonemap_hosts=K`.

## Export to colleagues / team reuse

Once debugging passes, send `dist/setup-ie-compat.bat` directly to colleagues in your company; they just run it as administrator (the same batch of legacy sites applies to everyone). This is the main use case: one person tunes it, the team reuses it.

- `.bat` files are often blocked by email / IM as high-risk attachments: temporarily rename it to `.txt` to send, then have the recipient rename it back, or use a USB drive / intranet share / corporate cloud drive.
- A forwardable pitch: "This is an internal config script; it needs to be allowed to run as administrator. It only adds our intranet sites to Edge's IE compatibility list and touches nothing else; just reopen Edge after running. Remember to save anything unsaved in Edge first."

### Verification

- Main check (for non-technical colleagues): just open that legacy system and see if it works — that's the only trustworthy verification.
- Advanced (optional): at `edge://compat/enterprise` check whether the version number incremented and the sites are listed; if it still doesn't take effect after reopening Edge, click **Force update** once.

## Domain GPO risk (the most common silent failure)

If your company's Edge policy is pushed centrally by IT through **Group Policy (GPO)**, the local registry policy written by this script **may be overridden or simply not take effect, and the script won't error out**. When you see "ran fine but the legacy system still won't open correctly", and at `edge://policy` the policy item already shows "Configured" yet still does nothing, ask IT to add the intranet sites to the Enterprise Mode Site List policy. The generated `.bat` also flags this at the end.

## Desensitization principle (important)

- This skill repository **contains no real intranet addresses / IPs**. Sites are supplied by you and written by the AI into `sitelist.xml`; placeholders always use reserved fake domains like `intranet.example.com:port`. ZoneMap registry entries are auto-derived by the generator from the list — don't hand-write IPs.
- Both `sitelist.xml` and `dist/` are **user data** (they contain real addresses) and are excluded by `.gitignore`; the only visible sample in the repo is `sitelist.xml.example`. **Never commit / publish a `.bat` or `sitelist.xml` containing real URLs to a code repository or the public internet** — but "not publishing" and "sending to colleagues" are two different things; you may send them to colleagues in your company.

## Notes

- Don't mix in the `edge://compat/SiteListManager` GUI to maintain the same list, or the schemas may overwrite each other.
- The one-time policies (Edge Enterprise Mode registry, Intranet zone, ActiveX) are already folded into the generated `.bat`; rerunning refreshes them, no separate maintenance needed.
- After deleting / adjusting sites, rerun `gen.py`; the generator first clears the old ZoneMap entries and rebuilds from the current list, so deleted sites are no longer mapped into the Intranet zone.
- Speak to non-technical colleagues in plain language ("which old IE engine should this legacy system use"); keep the technical labels internal — no need to throw them at the other person.
