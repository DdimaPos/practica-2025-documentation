# Guide on how to setup and run the project locally

## Prerequisites

- Nodejs v.22 or higher

## Set up

Clone the repo

```bash
git clone https://github.com/DdimaPos/practica-2025-students-forum.git
cd practica-2025-students-forum
```

Create your `.env` file.
For keys follow the steps described in [[Database Management#Environment Setup]].

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
DATABASE_URL=
```

Run then

```bash
npm i
npm run dev
```

Open the project on [localhost:3000](http://localhost:3000)

## Worflow process

To have a consistent workflow I've setup the following instruments:

- **prettier** - code formatter that rearranges the code according to configuration
- **eslint** - linter that parses the code to see the errors, bugs, stylistic errors, and suspicious constructs.
- **husky precommit and prepush** - utility to run eslint on each commit and pre-push to see if the branch matches the naming convention.

Branch naming will match one of the following:

- feature/your-task-name
- fix/your-task-name
- bug/your-task-name
- hotfix/your-task-name
- test/your-task-name

## Branching

Now we are making branches from `master` but later will work as following:

- Known branching system with a `master` and dev `dev`.
- Create your branches from `dev`
- If you have some merge conflicts when opening a PR solve them on your branch
