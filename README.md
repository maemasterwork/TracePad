# Tracepad: accounts, progress and a teacher dashboard

Tracepad is a static website (one `index.html`) that stores logins and progress in your own Supabase project.
Students and teachers sign in, every answer is saved to the student's account, students see their scores and
assignments, and teachers see each class.

What is in this folder:

| File | What it is |
| --- | --- |
| `web/index.html` | The whole app, with Lesson 1 inside. No build step. |
| `web/config.js` | The only file you edit: your Supabase URL and anon key. |
| `sql/schema.sql` | Tables, security rules and functions. Run once. |
| `sql/test_rls.sh`, `sql/shim.sql` | Optional: the security tests I ran (see "What was tested"). |

## Setup (about 15 minutes)

1. **Create a Supabase project** at supabase.com (the free plan is enough for a class). Pick a region near your students.
2. **Create the tables.** Open the project, go to *SQL Editor > New query*, paste all of `sql/schema.sql`, and press *Run*.
   It is safe to run again later.
3. **Choose your teacher invite code.** In the SQL Editor run (use your own secret):

   ```sql
   update public.app_settings set value = 'a-long-secret-phrase' where key = 'teacher_code';
   ```

   Anyone who types this code when creating an account becomes a teacher. Everyone else is a student.
   Until you change it, nobody can become a teacher.
4. **Sign-in settings.** Go to *Authentication > Providers > Email*.
   - If your students have no email address, switch **Confirm email** off. They then sign in with a username
     (stored as `username@students.tracepad.app`, which you can change in `config.js`).
   - If everyone has a real email, you can leave confirmation on. They will be asked to click the link first.
   - Under *Authentication > URL Configuration* set **Site URL** to the address where you host the site.
5. **Connect the site.** In *Project Settings > API* copy the **Project URL** and the **anon public** key
   into `web/config.js`. The anon key is meant to be public. Never put the `service_role` key anywhere in the site.
6. **Host the `web` folder.** Any static host works: Netlify (drag the folder onto app.netlify.com/drop),
   Cloudflare Pages, Vercel, or GitHub Pages. It needs HTTPS.
7. **First run.** Open the site, choose *Create account*, tick *I am a teacher*, and enter your code.
   Create a class. Give students the six-letter join code. They create accounts, join, and you will see them appear.

## How it works

- **Students** see *My progress*: assignments with due dates and status, scores per chapter, and the assessment score out of 90.
  Every prediction, quiz answer, exercise result, hint, checklist tick and "mark as read" is saved automatically.
  The chip at the top shows *Saved*, *Saving…* or *Offline, will retry*. Work done offline is kept on the device and sent later.
- **Teachers** see *My classes*: create classes, share join codes, open a class to get a table of every student by chapter,
  the class average, the concepts the class misses most, and per-student detail (every guess, quiz pick, exercise attempt and hints used).
  The *Assignments* tab sets a chapter and a due date and shows how many students have finished.
- **Security** is enforced by the database, not by the page. A student can read and write only their own rows.
  A teacher can read, but never change, the progress of students in their own classes. Students cannot make themselves
  teachers. The teacher code can only be read by the database functions.

## Things to know

- **Scores come from the student's browser.** The code runs in their browser, so a determined student could edit what is
  saved. This is fine for practice and formative checks. Do not use it for high-stakes grading without a proctored setting.
- **Password resets** are not built into the page. For email accounts use Supabase's reset flow; for username accounts,
  reset the password in *Authentication > Users*.
- **Removing a student:** delete their row in *Authentication > Users*; their progress is deleted with them.
- **Adding Lesson 2** later means adding its content and labs to the app. Nothing in the database has to change:
  progress is stored as small rows keyed by chapter number.
- **Privacy:** the database holds names, usernames or emails, and learning activity. Tell students and parents,
  and follow your school's rules. Supabase lets you pick the data region.

## What was tested

I could not test against your live Supabase project (I have no access to it). What I did test:

- `sql/schema.sql` was applied twice to a real PostgreSQL 16 database (with a small stand-in for Supabase's `auth` schema)
  and **32 security checks passed** (`sql/test_rls.sh`): students cannot read or write others' rows, teachers see only their
  own classes, nobody can promote themselves, the teacher code is unreadable, and anonymous users are blocked.
- The full app ran in a real browser against that database, using a stand-in for the Supabase client that executes
  real SQL with row-level security on. It covered teacher and student sign-up, wrong teacher code, wrong password, joining a class,
  saving and restoring progress on a second device, offline saving and catching up, the dashboard numbers, student detail,
  assignments, a student being blocked from teacher pages, Persian right-to-left layout, and progress reset.

The stand-in client is not the real `supabase-js`, so if something behaves differently on your project, the first things to check
are the Auth settings in step 4 and that `config.js` has the right URL and key.
