# Password Manager Migration Plan (LastPass → replacement)

Draft for morning review. Nothing here has been executed. I have no access to your accounts or devices, and none of your data should ever be pasted into this session.

## Your situation (assessed)

- LastPass account created around 2019, so it likely started at 100,100 iterations. It shows 600,000 now, raised after the 2022 theft.
- The master password was strong and unique.
- No crypto seed phrases or wallet keys were ever stored.

**Verdict: best case.** The stolen 2022 copy of your vault is very unlikely to be cracked. Treat migration and rotation as planned hardening, not an emergency.

**What is still exposed regardless:** the 2022 theft included unencrypted **website URLs**, plus your name, email, phone, and billing address. Attackers can read the list of sites where you have accounts (banks, brokerages, and so on) and can tie it to you. The main ongoing risk is **targeted phishing, impersonated support calls, and SIM-swap attempts**, not cracking. Phase D (section 6) is built around that.

## 0. Ground rules

1. **Never give me, or any chat or cloud tool, a password, vault export, or 2FA seed.** This session runs in a cloud container. All export and import steps happen on your own machine.
2. **Don't build our own manager.** Rolling your own crypto and vault storage is the worst security choice available. We adopt an audited, mature product. What we can build is tooling around the migration (see section 9).
3. **Moving the vault is not the same as being safe.** If your old vault was in the 2022 LastPass theft, the attacker has an encrypted copy that no migration affects. The real protection is **rotating the secrets inside it**, highest-value accounts first. Migration is step one, rotation is the actual fix.

## 1. Threat model (what we're defending against)

| Threat | Notes | Mitigation |
|---|---|---|
| Stolen encrypted vault copy (offline cracking) | Strength depends on the old master password and the KDF iteration count. Accounts created before about 2018 may have had very low iteration counts. Website URLs were not encrypted in the 2022 breach. | Rotate secrets, tiered by value. A strong new master passphrase protects only the new vault. |
| Server-side breach of the new vendor | Any cloud vendor can be breached. | Pick a design where a stolen vault is hard to crack (strong KDF, ideally a device-held secret key). |
| Phishing and credential stuffing (AI-assisted) | Likely your most realistic risk. | Unique passwords, passkeys and hardware keys, and autofill that only fires on the correct domain. |
| SIM swap / email takeover | Email is the master key to everything. | Hardware keys on email, carrier PIN, no SMS 2FA where avoidable. |
| Malware on your device | Defeats any manager. | OS updates, no pirated software, a separate browser profile for finance. |
| You locked out | Real risk of an over-secured setup. | Printed emergency kit, a second hardware key, a tested recovery path. |

## 2. Choosing the replacement (decision for the morning)

Verify current pricing, audit dates, and any incidents before committing. These are the options as I know them, and the facts may have moved.

| Option | Model | Strengths | Weaknesses |
|---|---|---|---|
| **1Password** | Cloud, paid | Two-secret key derivation (master password plus a device-held Secret Key), so a stolen server vault is far harder to crack. Strong UX, passkey support, good family and emergency features. | Closed source. Losing the Secret Key and the emergency kit is painful. |
| **Bitwarden** | Cloud, open source, free or cheap | Open source, independently audited, supports YubiKey 2FA, can be self-hosted. | Security rests on master password strength plus a configurable KDF. Set Argon2id and high settings. |
| **Proton Pass** | Cloud | Good if you already use Proton. Integrated email aliases. | Younger product. |
| **KeePassXC** | Local file | No vendor server to breach. Full control. | You handle sync (Syncthing or similar), backups, and mobile yourself. Easy to get wrong. |
| **Self-hosted Vaultwarden** | Your server | Control. | You become the ops and security team. Not recommended as a first step. |

**My recommendation:** Bitwarden or 1Password. If breach resistance is the top priority, I lean 1Password because of the Secret Key. If cost and open source matter more, Bitwarden with Argon2id and YubiKeys is solid. Skip self-hosting for now.

## 3. Phase A: Get the data out (can start tonight; ~30 min)

Do this on a **trusted, up-to-date personal computer**, not a work or shared machine.

1. Disconnect from cloud-synced folders. The export must not land in iCloud Drive, Dropbox, or Downloads sync.
2. Log in to LastPass in the browser. Go to Advanced options → Export. This produces a **plaintext CSV**. That file is the most dangerous thing you'll ever create, so treat it that way.
3. Save it to an encrypted volume (a VeraCrypt or macOS encrypted disk image) or a RAM disk. Name it something unremarkable.
4. Separately capture what the CSV misses:
   - Secure notes with attachments, and form-fill profiles. Open these manually.
   - Any **LastPass Authenticator** TOTP seeds. Migrate or re-enroll these in the new setup. Do not lose them.
   - Credit cards, bank details, and software licenses.
   - Any **crypto seed phrases** stored in the vault. If a seed phrase was ever stored there, treat it as compromised: create new wallets and move funds. See Phase D.
5. Do not delete anything in LastPass yet.

## 4. Phase B: Stand up the new manager (~1 hour)

1. Create the account on a clean device.
2. Master credential: a **6+ word random passphrase** generated with dice or the manager's generator. Never reuse a password from anywhere else. Memorize it, and write it on paper stored securely at home.
3. Print and store the **emergency kit** or recovery codes, offline, in two places.
4. Buy **two hardware security keys** (YubiKey or equivalent, one primary and one backup stored elsewhere). Enroll both on the manager.
5. Configure the KDF to the strongest setting available (Argon2id with high memory for Bitwarden).
6. Install the browser extension and mobile app. Disable "autofill on page load" and keep autofill limited to exact domain matches.

## 5. Phase C: Import and verify (~1 hour)

1. Import the CSV from the encrypted volume.
2. Verify: compare item counts, then spot-check 15–20 logins across categories, including old ones.
3. Re-create anything the CSV dropped (notes, attachments, cards).
4. **Securely destroy the CSV**: delete it and empty the encrypted volume. On SSDs, deleting the whole encrypted container, not just the file, is the reliable approach.
5. Keep LastPass logged in but untouched until Phase D progresses.

## 6. Phase D: Harden and rotate by tier (days to weeks)

The vault is very likely uncracked, so this is a measured cleanup. The priority is making the exposed list of sites useless to phishers.

| Tier | Accounts | When |
|---|---|---|
| **0: Keys to the kingdom** | Primary email(s), Apple ID or Google account, phone carrier (set a port-out PIN), domain registrar, the new password manager. Freeze credit at Equifax, Experian, and TransUnion (free). | First week |
| **1: Money** | Banks, brokerages, retirement, payment apps, credit cards, crypto exchanges. Move these to a **new private email or alias** that was never in LastPass and isn't public. | Weeks 1–2 |
| **2: Identity and recovery** | Government and tax sites, healthcare, insurance, cloud storage, social media, developer accounts such as GitHub, cloud providers and API keys | Weeks 2–4 |
| **3: Everything else** | Shopping, streaming, forums, one-offs | Over time. Delete accounts you no longer use. |

Review the vault once for items that can't be rotated and handle each:
- Regenerate 2FA backup codes and re-enroll any authenticator setup keys that were stored in notes.
- Change security-question answers to random values kept in the new manager.
- Note which ID numbers were stored (SSN, passport, license). The credit freeze covers most of the risk from those.

Phishing rule from now on: never act on an unexpected "security alert", email, text, or call. Open the app or type the site address yourself.

For each account:
- Generate a unique random password in the new manager.
- Switch 2FA to a hardware key or passkey where supported, and an authenticator app otherwise. Avoid SMS.
- Revoke active sessions and old app passwords or tokens.
- Update security questions with random answers stored in the manager.
- Check the account's recovery email and phone for changes.

Also rotate API keys and tokens, SSH keys if they were stored, and Wi-Fi and router credentials.

## 7. Phase E: Decommission LastPass (after Tier 0–1 are done)

1. Confirm the new vault is complete and you can log in on every device from a cold start.
2. Remove LastPass browser extensions and apps.
3. Delete the LastPass account. Deletion does not remove any stolen copy, but it removes your live exposure.
4. Remove LastPass entries from your authenticator and any recovery-email links.

## 8. Phase F: Ongoing hygiene

- Monthly: review the manager's breach and reuse reports.
- Quarterly: test your recovery path (emergency kit plus backup key).
- Set up emergency access for a trusted person, or write instructions for your household.
- Keep an encrypted offline backup of the vault export (updated twice a year, same handling rules as Phase A).
- Prefer passkeys for every site that offers them.
- Consider email aliases per site to resist credential stuffing and targeted phishing.

## 9. What I can build next (optional, runs locally on your machine)

- A small offline script that reads your exported CSV, **never prints or transmits passwords**, and produces a prioritized rotation checklist: tiering by domain, reused-password detection, weak-password flags, and dead or duplicate entries.
- A progress tracker (spreadsheet or checklist) for the rotation phase.

## 10. Remaining questions

Answered: account age (about 2019), master password (strong and unique), seed phrases (none).

1. Cloud or local? 1Password, Bitwarden, or KeePassXC? I recommend 1Password for its Secret Key design (see section 2).
2. Roughly how many items are in the vault?
3. How many devices and OSes (iPhone, Android, Mac, Windows, Linux)?
4. Family or shared vaults needed?
5. Budget for two hardware keys (about $50–$100 each)?

## Suggested timeline

| When | What |
|---|---|
| This week | Choose the product, order hardware keys, do Phases A–C, freeze credit, set a carrier PIN. |
| Weeks 1–2 | Tier 0–1, including the new private email for financial accounts. |
| Weeks 2–4 | Tier 2, then Phase E (delete LastPass). |
| Ongoing | Tier 3 and Phase F. |
