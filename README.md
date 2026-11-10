BRAINSPARK is a coding school platform that connects learners, tutors, schools and parents. Learners join a class with a code, learn HTML, CSS and JavaScript in a built-in coding space, read notes, hand in assignments and take quizzes. Tutors control what their class sees. Schools and the platform admin create and manage every account.

It runs with no server of its own: the web pages are one static file hosted on GitHub Pages, and Supabase provides the database, login and security rules.

## What each role can do

| Role | Can do |
|---|---|
| Admin | Add schools, create any account, set grades, reset passwords, suspend or activate people, see all classes |
| School admin | Create learners and tutors for their own school, set grades, reset passwords, create classes, see their school only |
| Tutor | Create classes and share the class code, choose the coding space (HTML, CSS, JavaScript, or a mix), post announcements and notes, upload files, build courses and lessons, set assignments, grade work, build quizzes |
| Learner | Join a class with a code, read lessons and notes, code in the coding space, hand in assignments, take quizzes, see grades and feedback |
| Parent | Account exists, dashboard not built yet |

Nobody can sign themselves up. Accounts are created by an admin or a school, and a learner's grade and school are set by them, never by the learner.

## How it is built

- **Web pages:** one file, `index.html` (plain HTML, CSS and JavaScript), hosted on GitHub Pages
- **Database and login:** Supabase (PostgreSQL with row level security)
- **Account creation:** one Supabase Edge Function, because creating logins needs a secret key that must never be in a web page
- **Coding space:** learner code runs inside a sandboxed frame, so it cannot touch the rest of the page

Security rules live in the database, so the web page cannot bypass them. Tutors only reach their own classes, schools only their own school, and learners only their own work.

## Files

```
index.html                                  the whole web app
README.md
supabase/
  migrations/
    001_schema.sql                          tables, security rules, functions
    002_sample_course.sql                   optional starter Python course
    003_class_space.sql                     coding space settings, notes, file uploads
    004_admin_accounts.sql                  admin-created accounts, grades
    005_default_admin.sql                    the first admin account
    006_assignments_quizzes.sql             quizzes, grading notifications
  functions/manage-accounts/index.ts        the account creation service
```

## Set up from scratch

1. **Create a Supabase project** at supabase.com. The free plan is enough to start. Keep the database password safe.
2. **Turn off email confirmation:** Authentication, then Sign In / Providers, then Email. Turn **Confirm email** off. Logins use usernames, so there is no inbox to confirm.
3. **Run the SQL files** in the SQL Editor, one at a time, in this order: `001`, `002` (optional), `003`, `004`, `005`, `006`. If one fails, stop and read the error before running the next.
4. **Deploy the account service:** Edge Functions, then Deploy a new function. Paste in `functions/manage-accounts/index.ts`. Turn **Verify JWT** off, because the function checks the signed-in user itself. After deploying, look at the function's URL in the list. The last part of the address is its name. Supabase may give it a random name such as `swift-task`.
5. **Set the same name in `index.html`:** near the top of the script, change `ACCOUNTS_FN` to that name.
6. **Check the other two settings at the top of the script:** `SB_URL` and `SB_KEY` are your project URL and publishable key (Project Settings, then API). The publishable key is meant to be public. Never put the service role key in the web page.
7. **Turn off public sign-ups:** Authentication, then Sign In / Providers, and switch **Allow new users to sign up** off.
8. **Publish:** put `index.html` in a GitHub repository and turn on Pages (Settings, Pages, deploy from the main branch, root folder).

Usernames are turned into a hidden email address, `username@erp.local`, because Supabase login needs an email. The same text is set in `EMAIL_DOMAIN` in `index.html` and `DOMAIN` in the Edge Function. Keep them identical.

## First sign-in

`005_default_admin.sql` creates one admin with username `admin` and password `admin123`. **Change it immediately.** The page forces a new password on first sign-in, but until then anyone who knows the default can get in, so do this the moment the SQL has run.

Then, as admin:
1. **Schools:** add a school, then create its school administrator.
2. **Accounts:** create tutors and learners. Learners are entered as `Name, grade`, one per line. The system generates usernames and first passwords, shows them once, and offers a CSV download to print or share.
3. Everyone must choose a new password at their first sign-in.

## Day to day

**Tutor:** open My classes, create a class, and share the code. Open the class to set the coding space, post an announcement, add notes and files, assign a course, create assignments and quizzes, and grade work.

**Learner:** sign in, then open the class (or use Join a class with a code). Work through lessons, use the coding space, hand in assignments, and take quizzes.

**Lost password:** an admin or school admin resets it from Accounts. Email reset does not work for these accounts.

## Troubleshooting

| Message | Cause and fix |
|---|---|
| Failed to fetch when creating accounts | The page cannot reach the Edge Function. Check `ACCOUNTS_FN` matches the function's real name, the function is deployed, and Verify JWT is off |
| column ... does not exist | A SQL update has not been run. Run the missing file from the list above |
| Wrong username or password | Usernames are not case sensitive. Ask an admin to reset the password |
| Supabase rejected the email address | Change `EMAIL_DOMAIN` (and `DOMAIN` in the function) to another domain. This affects only new accounts |
| Page looks old after an update | Hard refresh with Ctrl+Shift+R |

## Known limits

- Only HTML, CSS and JavaScript are supported in the coding space. Python and Scratch are not built yet.
- The parent dashboard, attendance, timetables, notification bell and reports are not built yet.
- A quiz cannot be edited. Delete it and create a new one.
- On Supabase's free plan, a project that is not used for about a week is paused, and there are no automatic backups. Use the dashboard's backup or export option regularly if the data matters.
- All the web code is in one file. That is simple to host but harder to maintain as the system grows.

## Handling children's data

This system stores names, grades and school work for children. Before using it with real learners, get the school's agreement, tell parents what is stored, and check what Kenya's Data Protection Act requires of you as the operator.

## Planned next

Parent dashboard, attendance, notifications, reports, auto-checked coding challenges, Python in the browser, weekly backups and an admin activity log, then a rebuild of the front end in React and TypeScript with tests.
