# scrumfaster

* the real scrum master
* automates GitHub issues and projects

## what does this do

* `scrumfaster` takes a markdown or HTML input, parses it based on some made up rules, and creates GitHub project draft items or issues
* as you're using a markdown editor, you can rearrange, duplicate, or remove cards quickly with your familiar key bindings
* you will have more productive planning sessions as you'll spend less time clicking on text boxes
* and moving between cards if you decide to break down work further or change something

## example

for the following markdown:

```markdown
# my cool board

## Sprint 1

* [ ] Profile avatars
    * [ ] Create database migration for avatar field [@itsjfx] [status=Done] [labels=database] [1]
        * Name the field `avatar` in the `users` table
        * Set value for existing users to https://...
    * [ ] Accept avatar parameter in `update_user` API call [@itsjfx] [labels=api] [1]
        * Use existing image upload mechanisms
        * Limit image size to 10mb
    * [ ] Display and allow updating avatars on frontend [labels=frontend] [2]
        * Only display avatars on the users public profile page
        * Thumbnails aside comments to be implemented in later card
* [ ] Dark mode
    * [ ] Add ui toggle for dark mode [labels=frontend] [1]
        * Store preference in local storage
    * [ ] Implement styles [labels=frontend] [2]
        * Apply styles dynamically based on user preference
* [ ] Delete jeff from database [labels=database] [1]
```

the example is [provided in the repository](./example.md) if you'd like to run `scrumfaster` against it

### adding issues

running with `scrumfaster --import-issues -f example.md --owner=OWNER --repo=REPO --project-id X`:

1. `# my cool board` will be ignored (this is not supported at the moment)
2. a milestone named "Sprint 1" will be created if it does not exist
3. a card named "Profile avatars: create database migration for avatar field" will be created
    * it will be assigned to `itsjfx`
    * it will have status `Done`
    * it will have the `database` label
    * it will have `Points` of `1`
    * it will have body:
        + Name the field `avatar` in the `users` table
        * Set value for existing users to https://...
    * it will be under milestone "Sprint 1"
4. it'll continue doing the above until it's complete or it crashes

here's how it looks like in GitHub:

### adding draft items

running with `scrumfaster --import-drafts -f example.md --project-id X --milestone-field epic`:

1. `# my cool board` will be ignored (this is not supported at the moment)
2. a card named "Profile avatars: create database migration for avatar field" will be created
    * it will be assigned to `itsjfx`
    * it will have status `Done`
    * it will **NOT** have the `database` label, as labels are not supported for draft items
    * it will have `Points` of `1`
    * it will have body:
        + Name the field `avatar` in the `users` table
        * Set value for existing users to https://...
    * it will **NOT** be under a milestone as this is not supported for draft items
    * however, as `--milestone-field epic` was set, and `Epic` was pre-populated, it will set the milestone value under the `Epic` field
3. it'll continue doing the above until it's complete or it crashes

## usage

```
$ scrumfaster --help
usage: scrumfaster [-h] [-f FILE] [-l {debug,info,warning,error,critical}] {import-issues,import-drafts} ...

positional arguments:
  {import-issues,import-drafts}
    import-issues
    import-drafts

options:
  -h, --help            show this help message and exit
  -f, --file FILE       Markdown file or HTML file to parse (default: stdin)
  -l, --log {debug,info,warning,error,critical}
                        Logging level (default: info)

$ scrumfaster import-issues --help
usage: scrumfaster import-issues [-h] --owner OWNER --repo REPO [--project-id PROJECT_ID]

options:
  -h, --help            show this help message and exit
  --owner OWNER
  --repo REPO
  --project-id PROJECT_ID

$ scrumfaster import-drafts --help
usage: scrumfaster import-drafts [-h] --project-id PROJECT_ID [--milestone-field MILESTONE_FIELD]

options:
  -h, --help            show this help message and exit
  --project-id PROJECT_ID
  --milestone-field MILESTONE_FIELD
```

* `import-issues`
    * creates issues
    * if you specify `--project-id`, it will create items in a GitHub project linked back to the issues
    * and will set any additional fields on the project cards (e.g. `status`)
    * i suggest not enabling any workflow automations to set status or automatically create items out of issues in your GitHub project
    * as GitHub's project workflows will not create the items in correct order, but `scrumfaster` will
* `import-drafts`
    * only creates draft items on a project
    * if you specify `--milestone-field KEY`, it will populate the field key with the current milestone for the ticket

## when to use issues or drafts?

* draft items are great if you don't want to flood your project with issues
* in open-source projects, people unfortunately correlate issues == problems, even if issues are created by maintainers for tracking feature development
* drafts are also simpler, if you have a project with 1 - 2 people, it may be sufficient
* however, issues are much more powerful. if you want to scale, i suggest creating issues
* with issues you can:
    + assign labels
    + reference issues in other issues
    + reference issues in your commit messages or PRs, and everything is tracked within the issue
        * this is the most useful feature due to the audit trail
        * e.g. referencing a ticket in your commit message `#1` will add it to the issues log
        * you can also close issues by including `implements #1` in your commit message, it will mark as completed and link to your commit
        * <https://github.blog/news-insights/product-news/closing-issues-via-commit-messages>
    + use milestones
    + maintain history, closed issues are always visible
* there's probably more things, i've only recently began using issues

## rules

1. you can create issues, issues AND project items, or draft items
    * to create only issues: `import-issues --owner OWNER --repo REPO`
    * issues and project items: `import-issues --owner OWNER --repo REPO --project-id X`
    * only draft items: `import-drafts --project-id`
2. each `<h2>` or ## heading is a "milestone" or "milestone field"
    * when creating issues
        + if a milestone does not exist, it will be created
        + this value has case insensitive lookups (so a milestone named "Sprint 1" and "sprint 1" are equivalent)
        + if a milestone needs to be created it will preserve the casing from the heading
    * when creating draft issues
        * milestones are ignored as you cannot associate milestones to project items
        * however, you set `--milestone-field FIELD_NAME` to associate the milestone to a field
        * the field has to exist prior, and if you want to use a single select field the values must be pre-populated correctly
3. if there are nested items, a create a card will be created for each leaf/lower item, and the name will be concatenated based on its parents
    a. i may change this to create subtasks for issues
4. cards can have fields, captured in brackets [] -- any keys referenced are case-insensitive
    a. if it starts with @, then its assigning a list of people to a ticket
        * e.g. [@itsjfx, somebodyelse, andsomeoneelse]
    b. if its a number, then its shorthand syntax to set the value of a field named `points` (case insensitive)
        * if `points` does not exist on the project, this will crash
    c. if there's an equals, then its setting the value of a field key directly
        * e.g. `status=Done` will set the value of a field named `status` to `Done`
        * or `labels=frontend, backend` will assign `frontend` and `backend` labels on the ticket
    d. if its a single select field, the value will be looked up, if its an invalid option the program will crash
    e. theres no way to escape these brackets at the moment
5. if a task item has a nested unordered list (* or -) or an ordered list (1.), it will be used in the body of the card
    * if the list contains a single item, it will be used as the body directly (no list or list item will be displayed)
    * otherwise, the body will be the unordered list or ordered list
6. project items will default to `Todo`, unless overridden by setting `status=X`

## authentication

* To authenticate with GitHub you can set the `GITHUB_TOKEN` environment variable, or it will attempt your credentials from the `gh` CLI
* you may need to allow some additional scopes for projects or issues

## dependencies

* python: ideally use `uv` or the [requirements.txt](./requirements.txt)
* and you need the wonderful [pandoc](https://pandoc.org) somewhere on your path to convert to/and from markdown

## TODO

* figure out what scopes are needed for auth
* better logging ...?
* support specifying data for epics when creating them? e.g. due dates
* support alternatively to `pandoc` (what are they)?
* refactor into multiple files
* pypi
