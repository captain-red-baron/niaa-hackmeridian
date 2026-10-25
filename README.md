# Niaa Pay on Stellar

HackMeridian, Lisbon, 25–26 October 2026.

Niaa is an app to back the people who get things done. A sponsor backs a changemaker's activity, and every amount can be traced from the sponsor to the result. Niaa Pay on Stellar holds the sponsor's money in a Trustless Work escrow on Stellar and pays it out milestone by milestone, once the sponsor approves the proof.

## The flow in four steps

1. **Commit.** The sponsor commits an amount, a reporting plan and the conditions in one step.
2. **Accept.** The changemaker accepts and signs; a hash of the agreed terms goes on chain. The sponsor then locks the budget in the escrow.
3. **Submit.** The changemaker submits proof for a milestone. The proof files are hashed at upload and the hash goes on chain.
4. **Approve and pay out.** The sponsor approves, and the escrow releases the milestone in USDC to the changemaker's wallet.

Both sides get a notification at every step, so each one knows when it is their turn.

## What goes on chain

Only hashes, neutral ids, escrow amounts and wallet addresses. Never names, emails, file contents or other personal data. The agreement hash is SHA-256 over the canonical JSON (RFC 8785) of the agreed terms. Anyone can check a proof on the public Verify page: drop the photo, the browser hashes it locally and matches it against the milestone.

## Platform fee

Niaa's fee is 10% of the sponsor's own contribution, paid on top. The sponsor locks contribution plus fee, the changemaker receives 100% of the contribution, and the fee leaves through the escrow. The fee is never charged on the whole activity budget: a sponsor who gives 2,300 of a 5,000 activity pays 230.

## Built with

- [Stellar](https://stellar.org) testnet and [`@stellar/stellar-sdk`](https://github.com/stellar/js-stellar-sdk)
- [Trustless Work](https://www.trustlesswork.com) v2 multi-release escrows
- USDC on Stellar
- Browser signing with [Stellar Wallets Kit](https://github.com/Creit-Tech/Stellar-Wallets-Kit) (Freighter)
- Next.js, Clerk, Supabase

Testnet only. All money in the demo is test money with no real value.

---

# Demo guide

Everything happens live in the browser on Stellar testnet, from fresh accounts. Use two browser windows (or two laptops), each with its own browser profile:

- **A = sponsor** (in the story: Inês, for a foundation)
- **B = changemaker** (in the story: Amani, a community organiser near Nampula, Mozambique)

App: `https://niaa-git-staging-jaden-data.vercel.app`

## 0. Prepare

1. Install [Freighter](https://www.freighter.app) in both browser profiles, open it once in each, and set the network to **Testnet**. You need two Freighter accounts, one per profile.
2. Make two test accounts. Sign-up email `<name>+clerk_test@example.com` (for example `amani+clerk_test@example.com` and `ines+clerk_test@example.com`), verification code `424242`.
3. B (changemaker): at "How will you use Niaa?" choose "I am a changemaker". Pick a theme ("Which ... fits your work?") and remember it.
4. A (sponsor): choose "I am a sponsor" and enter an institution name (for example "Fundação Esperança"). Pick the **same theme** as B. The theme decides which activities a sponsor sees.
5. Wallets. No external faucet is needed. Under the linked wallet, a panel **"Get ready for USDC"** walks through three buttons, in this order:
   1. **"Activate on the test network"** (only for a brand-new account): gives it a little test XLM for network fees. Shows "Account activated. Next, allow USDC."
   2. **"Allow USDC"**: Freighter asks you to confirm once (the USDC trustline). Then "Your wallet is ready for USDC."
   3. **"Add test dollars"**: enter an amount in "Test dollars to add" (up to 1,000,000 at a time), then click the button. Shows "Test dollars added." and the balance.
   - **B (changemaker):** `/cm/budget/payouts`, "Payout wallet", "Connect a wallet", pick Freighter. Then "Activate on the test network" if shown, and "Allow USDC". B needs no test dollars. Do this before accepting: B cannot accept while the payout wallet cannot receive USDC.
   - **A (sponsor):** `/funder/payment-settings`, "Connect a wallet", pick Freighter. Then "Activate on the test network" if shown, "Allow USDC", and "Add test dollars" with at least the amount you will lock (100 is plenty for the demo amount below).

A Stellar Passkey wallet can hold and receive USDC but cannot sign here, so use Freighter on both sides.

If it goes wrong: "This account isn't active on the network yet. Activate it below." or "This wallet can't receive USDC yet. Allow USDC below." means the panel's next button is waiting. "The test network did not answer." means try again in a moment.

## 1. Changemaker creates the activity

- Who: B. Where: `/cm/activities/new?mode=form` (or "Use the form instead" on the first screen; the default is a voice interview with Niaa, which needs a microphone).
- Required: Title, Description, theme, "I want more of", "I want less of", "Compass indicator" and "Unit". For funding choose "Funding wanted" and fill "Budget needed". There is no location field, so put the place in the Description.
- Paste this:

| Field                        | Value                                                                                                                                                                                            |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Title                        | Three wells for villages near Nampula                                                                                                                                                            |
| Description                  | Amani is a community organiser near Nampula, in northern Mozambique. We drill three wells for three villages, so families stop walking hours for water. A local crew drills and tests each well. |
| Theme                        | the theme you picked in step 0                                                                                                                                                                   |
| I want more of               | Families with clean water within ten minutes of home. Time for children to be in school.                                                                                                         |
| I want less of               | Hours spent walking to fetch water. Illness from unsafe water.                                                                                                                                   |
| Concrete ideas (optional)    | Site survey and community meeting. Drill and test the first well. Drill the second and third wells.                                                                                              |
| Compass indicator            | Wells drilled and tested                                                                                                                                                                         |
| Unit                         | Count (#)                                                                                                                                                                                        |
| Funding                      | Funding wanted                                                                                                                                                                                   |
| Budget needed                | 10000                                                                                                                                                                                            |
| Start date (optional)        | a date in the next two weeks                                                                                                                                                                     |
| Why I want to do this (opt.) | My village walked four hours a day for water when I was a child.                                                                                                                                 |
| Why I can do this (opt.)     | I have organised two community projects and know the local drilling crews.                                                                                                                       |

- Shorter alternative: Title "Solar lamps for a night school"; Description "A night school in a village near Beira has no light after six. We buy 40 solar lamps and a charging station."; "I want more of": "Evenings with light to study."; "I want less of": "Candles and kerosene."; Compass indicator "Lamps in use", Unit "Count (#)"; Funding wanted; Budget 800.
- Click "Create activity". The page shows "Your activity is created". There is no separate publish step.

If it goes wrong: "Some fields need your attention" highlights the first bad field. Type the budget without thousands separators (`10000`).

## 2. Sponsor commits

- Who: A. Where: `/funder/activities`. Open "Three wells for villages near Nampula" (filter by theme or "Awaiting funding" if needed) and click "Sponsor this activity".
- "Your contribution": the activity asks for 10,000, but on testnet keep the live amount small: commit **8**. With the Detailed plan the sponsor locks 8.80 (8 plus the 10% fee). The summary shows "Your contribution" (100% to the activity), "Niaa platform fee (10%)" and "Total you pay".
- Reporting plan:
  - **Detailed**: add deliverables whose amounts add up to your contribution, for example "Site survey, first well drilled and tested" and "Second and third wells drilled and tested".
  - **Simple**: one final report and one final milestone holding the whole amount, no platform fee.
- Conditions: leave the defaults (USDC, "Niaa (default)" settles disputes).
- Click "Review commitment", then "Commit funding". You see "Funding committed" and a link to the sponsoring.
- Other side: B's bell shows the new commitment. A sees "Waiting for <changemaker> to accept".

If it goes wrong: the activity is not in the list = the themes differ (step 0). With Detailed, the milestone amounts must add up to the contribution.

## 3. Changemaker accepts

- Who: B. Where: the bell, or `/cm/opportunities`, tab "Commitments from sponsors". Click "Review and accept".
- Read the conditions, click **"Accept and start the project"** and sign with Freighter. The terms hash goes on chain; the project starts only once that signature is recorded.
- Other side: A's bell says the project has started, with the next step "Lock the budget". B sees "Waiting for the sponsor to lock the budget."

If it goes wrong: "Your payout wallet isn't ready for USDC yet." = finish "Activate" / "Allow USDC" (step 0, item 5). "Your current wallet can't sign" = make Freighter the current wallet.

## 4. Sponsor locks the budget

- Who: A. Where: the bell, then the sponsoring (`/funder/funding/<id>`).
- Click **"Lock the budget"**. Freighter asks for two signatures back to back: create the escrow, then put the budget in (contribution plus fee). Approve both.
- Result: "Held safely" on both screens. Open the escrow in the Trustless Work viewer.
- Other side: B's bell says the escrow is funded.

If it goes wrong: only the first signature went through = the card shows "Step 2 of 2: put the budget in"; click "Put the budget in the escrow". "Your wallet doesn't hold enough USDC for this budget." = "Add test dollars" under Payment settings. "The Changemaker's payout wallet can't receive USDC yet." = B does "Allow USDC". Freighter on mainnet = switch it to Testnet.

## 5. Changemaker submits proof

- Who: B. Where: `/cm/deliverables`, or the bell. Click "I'm done – submit for review".
- Upload two photos (hashed at upload) and a short note, then sign with Freighter to put the proof on chain.
- Other side: A's bell asks to review the proof, then approve and pay out.

If it goes wrong: submitting is only possible after the project started and the escrow is funded (steps 3 and 4).

## 6. Sponsor approves and pays out

- Who: A. Where: the bell, then the sponsoring. Click **"Approve and pay out"**.
- Choose the outcome ("Succeeded"). For the last milestone tick "This is the final milestone"; otherwise define the next milestone. Confirm and sign in Freighter (approve, then release).
- Other side: B's bell: "USDC arrived". B receives 100% of the milestone amount.
- Show it: open the transaction on [stellar.expert](https://stellar.expert/explorer/testnet) from the sponsoring page. Then open the public Verify page (`/verify/<agreementId>`) in a signed-out window and drop a proof photo: the browser hashes it locally and matches the milestone.
- With two milestones, repeat steps 5 and 6 for the second one, or stop after the first to show "Planned, not paid out".

If it goes wrong: the payout is not in B's wallet yet = wait a few seconds and reload; testnet can lag. Approved but not paid = click "Pay out" on the milestone.

Keep a screen recording as a fallback in case testnet is slow.
