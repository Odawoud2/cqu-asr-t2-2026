# Assessment Sample Register — setting it up

One term only, Term 2 2026, retired after the GAC visit. Nothing here needs
CQU Digital Services. Budget about 25 minutes.

---

## 1. Create the Firebase project

Create a project in the Firebase console, for example `cqu-asr`. Turn Google
Analytics off.

## 2. Firestore

1. Build → Firestore Database → Create database.
2. Choose **australia-southeast1 (Sydney)**, so student work stays in Australia.
3. Start in **production mode**. The rules are replaced in step 5.
4. Note the database name. If it is anything other than `(default)`, set
   `databaseId` in the page to match, or writes fail without saying why.

## 3. Cloud Storage

1. Build → Storage → Get started, same Sydney region.
2. This is the step needing the **Blaze (pay-as-you-go)** plan, so a card has to
   go on the project.

   For one term this is effectively free. The 28 real files collected from
   PPMP20007 and PPMP20008 averaged 3.1 MB per grade band, which puts a full
   term across all 36 written assessments at roughly 380 MB and about 330
   files. Blaze includes 5 GB, so a single term sits well inside the free
   allowance.

3. Set a budget alert at $5/month regardless. Google Cloud console → Billing →
   Budgets & alerts. This is the safety net on an account with a card attached.

   Files are stored whole, so a download is a straight fetch of each file. That
   is the reason for choosing this over keeping files in Firestore, where a
   1 MiB document limit would have split the largest submission we measured
   (12.1 MB) across 17 chunks and made every download depend on reassembling
   them perfectly.

## 4. The two access codes

1. Build → Authentication → Get started → enable **Email/Password**. Leave
   email link off.
2. Add two users under the Users tab. The addresses are never emailed, so they
   can be anything; these are what the rules expect:

   | Address | Password | Who gets it |
   |---|---|---|
   | `asr-coordinator@register.local` | your coordinator code | the ten unit coordinators |
   | `asr-admin@register.local` | your administrator code | you |

3. Make both codes long. Twelve characters or more, and not a word. Anyone with
   the coordinator code can read every file in the register, so it is worth
   treating like a door key rather than a convenience.
4. Settings → Authorized domains: add the domain you will host on.

Sign-in is the code alone, with no name asked for. Because the code is shared,
the rules cannot tell one coordinator from another; a coordinator identifies
themselves by picking their own name from the dropdown at the top of the page,
which is what scopes the table to their units. That is a convenience, not proof
of identity, and anyone with the coordinator code can switch to any name.

## 5. Rules

Paste `firestore.rules` into Firestore → Rules and `storage.rules` into
Storage → Rules, then publish both. If you used different addresses in step 4,
change them at the top of both files.

What the rules do:

- Nothing is readable or writable without one of the two codes.
- A coordinator can add and update, but cannot delete.
- Only the administrator code can delete a stored file or change a unit record.
- Uploads are capped at 64 MB per file.

## 6. Fill in the page

Open `asr-app.html` in a text editor. Near the top of the first
`<script type="module">` block is a `CONFIG` object:

- `firebase` — copy the whole config from Project settings → Your apps → Web
  app. Register a web app first if there isn't one.
- `databaseId` — from step 2.
- `term` — leave as `"T2-2026"`.
- `accounts` — only change these if you used different addresses in step 4.

The Firebase config is not a secret and is meant to be in the page. What keeps
people out is the rules plus the codes.

## 7. Deploy

Currently deployed from GitHub Pages at
<https://odawoud2.github.io/cqu-asr-t2-2026/>, served from `main` at the
repository root as `index.html`. A push to `main` republishes within a minute
or two.

`odawoud2.github.io` is already in Authentication → Settings → Authorized
domains. Any other host has to be added there too, or sign-in is refused.

## 8. First run

1. Open the page, choose **Course administrator**, enter the administrator code.
2. Press **Set up this term**, once. It asks for `register_data.json`; pick it
   from wherever you saved it. That file holds the twelve units, forty-five
   assessments, and the instruction text, rubric files and quiz links already
   collected from Moodle.

   The register content is deliberately not built into the page. The page is on
   a public address, so anything inside it could be read from the source before
   sign-in; keeping it in a file you supply once means the instruction text only
   ever exists behind the codes.
3. The banner should turn green and read **Live**.
4. Send the coordinator code round with the link.

---

## The banner is the thing to watch

The page refuses to write unless it has first loaded the register from Firestore
in that session. A red **Read-only** banner means you are looking at the
built-in copy and nothing you do will be saved.

This is deliberate. The failure that cost you data in PLAC was a save running
while the page held fallback data, which overwrote the live database. Here a
failed read can only ever produce a refusal.

## Where things are stored

```
Firestore   terms/T2-2026/units/PPMP20008
            terms/T2-2026/units/PPMP20008/assessments/AT3

Storage     T2-2026/PPMP20008/AT3/Instructions/...
            T2-2026/PPMP20008/AT3/Rubric/...
            T2-2026/PPMP20008/AT3/Assessment Samples/HD/...
```

File bytes go to Storage; Firestore holds only names, sizes and paths, so no
document approaches the 1 MB limit.

## Take a copy as you go, not once at the end

The register is a single copy until it is downloaded. Run **Save files to my
computer** every week or two from the first upload onward, and keep the folder
on your own machine. A failure anywhere then costs you the last fortnight
rather than the term.

The page helps with this. When you are signed in as administrator the green
banner shows how long it has been since the last full download, counted on that
device. After a fortnight it turns amber. If it says *never*, nothing has been
taken off the server yet.

## Getting everything out, before you shut it down

**Save files to my computer** pulls every stored file back out and writes the
folder structure to a folder you pick:

```
PPMP20008/AT3/Instructions/PPMP20008_AT3_Instructions.txt
PPMP20008/AT3/Rubric/PPMP20008_AT3_Rubric.txt
PPMP20008/AT3/Assessment Samples/HD/PPMP20008_AT3_HD.docx
PPMP20008/AT3/Assessment Samples/HD/PPMP20008_AT3_HD_Feedback.xlsx
PPMP20008/AT3/PPMP20008_AT3_Notes.txt
```

Every piece of typed text becomes a text file in the folder it belongs to:
instruction text, rubric text, Moodle links, a band's written feedback, and the
moderation notes for the assessment. Bands with nothing in them get no folder.

Chrome and Edge write straight into the folder you choose. Other browsers give
you a zip with the same structure.

## Retiring it after GAC

Do these in order, and do not skip the verification.

1. Sign in as administrator and run **Save files to my computer**.
2. Run **Export to Excel** as well, for the register itself.
3. Open the folder and check the file count and total size against what the
   summary row claimed. This is the step people skip.
4. Take a second copy somewhere that is not the same disk.
5. Only then: Firebase console → Project settings → delete the project, and
   delete the GitHub repository.

Deleting the Firebase project destroys the files permanently and there is no
recovery, so step 3 matters more than it looks. Keep the card on the account
until the project is actually gone.
