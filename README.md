# FLOÉ Joy Arcade — GAME_BOOTH

Production-oriented booth minigames: 20-card memory matching and 3D plush claw machine.

## Gameplay
- Preview: 6 seconds, followed by 30 seconds of matching with unlimited attempts.
- 5+ pairs: one claw drop. 3–4 pairs: two candies at booth. 0–2 pairs: consolation.
- 24–28 randomized plush toys; the claw automatically holds a toy when aimed correctly and dropped.
- One staff-issued code allows one start. Refreshing or changing devices cannot reuse a code.

## Infrastructure
- Source: https://github.com/biliz-nguyen/GAME_BOOTH (main)
- Vercel: floe-game-booth (automatic deployment on main)
- Supabase: project etozewzatjchbyrgtorw (GAME_BOOTH)
- Edge Function: https://etozewzatjchbyrgtorw.supabase.co/functions/v1/arcade-entry
- Client begins with {action:'begin',code:'FLOE-...'} and finishes with {action:'finish',token,matches,caught}.

## Staff instructions: export 100 existing single-use codes
Open Supabase project etozewzatjchbyrgtorw -> SQL Editor. Run the following query and export as CSV:
```sql
select sc.code, case when ec.activated_at is null then 'UNUSED' else 'USED' end as status, ec.activated_at, ec.finished_at
from arcade_private.staff_codes sc join arcade_private.entry_codes ec using(code_hash)
order by sc.created_at, sc.code;
```
Give each customer exactly one unique code. Do not commit these private codes to GitHub.
To create 100 more codes, run:
```sql
select arcade_private.issue_staff_codes(100);
```

## Security / prize limitations
- Single-use participation is enforced transactionally in Supabase, but one PERSON = one code requires staff checking distribution.
- An opened play code remains used even if the guest closes the page.
- Browser-generated gameplay results can be manipulated. Database only records pending staff verification; it does not issue prizes or decrement stock yet.
- Prize stock is preconfigured as 100 vouchers / 50 keychains / 1 handmade keychain, but automated claims are disabled pending authoritative validation.
- Never expose service_role or other secret keys in frontend or public GitHub.
- Private tables are protected with RLS and blocked for anon/authenticated database roles.
- Before unsupervised production rollout: implement server-validated rewards, rate limits, and a staff redemption interface.

## Shared QR Preview: redemption at the counter

Preview is deployed from branch `preview/single-qr-mobile` and is separate from `main`. A browser can start only once (device-local storage + an atomic Supabase row). This is **not identity verification**; clearing browser storage or another browser can create a new attempt.

After a successful catch (or 3–4 matching pairs for candy), the game calls the `arcade-guest-preview` Edge Function which issues a unique **pickup reference** (`J` + 10 hex characters). It is recorded in `arcade_private.guest_browser_attempts.claim_code` and cannot be reissued for the same attempt.

The staff web page is at **`/staff.html`** on the Preview URL. Staff log in using **a separate Supabase Auth email/password account**. No public user or service credentials appear in the app. At present, no authorized staff account is configured, so the redemption UI cannot be used until the following setup is completed.

### One-time staff account setup
1. In Supabase **Authentication → Users → Add user**, create an account for the FLOÉ booth staff with a strong private password.
2. In **SQL Editor**, run the following (replace the email with the email of the account you just created):
```sql
insert into arcade_private.staff_members(user_id)
select id from auth.users where email = 'REPLACE_WITH_STAFF_EMAIL'
on conflict(user_id) do update set enabled=true;
```
3. Open `/staff.html`, sign in, enter the 11-character pickup reference shown on the guest phone, and press **Kiểm tra mã**.
4. Check the guest is actually at the booth and their game result is plausible. Select the **actual** prize being handed to them, then press **Xác nhận đã trao quà và trừ kho**.
5. Database validates eligibility, stock and an unclaimed result, then decrements exactly one item. Subsequent attempts to redeem the same code are refused atomically.

### Stock and redemption reporting
```sql
select prize_id,title,stock from arcade_private.prize_stock order by prize_id;
select claim_code,prize_claimed,claimed_at from arcade_private.guest_browser_attempts
where claimed_at is not null order by claimed_at desc limit 100;
```

**Important:** Claim results originate from the browser and can be forged; a code is **not a cryptographic attestation that the guest actually won**. Staff should visually check the game flow before handing out valuable prizes. The 1 QR demo does not include user identity verification or server-authoritative game scoring. Do not treat it as fraud-proof reward distribution.
