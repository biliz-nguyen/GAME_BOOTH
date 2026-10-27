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
