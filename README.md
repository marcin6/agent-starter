# Agent starter: instructions and skills for a frontend/fullstack project

[Wersja polska poniżej](#polski)

A universal set of instructions for coding agents (Claude Code, Codex and any other tool that reads `AGENTS.md`). The default stack is Next.js App Router + TypeScript + Tailwind + Zod + Vitest, with a separate backend (CMS/API) and optional payments.

This README describes the starter itself. Don't copy it into a new project.

## What's inside

```
AGENTS.md                      main instructions template (to fill in)
.claude/skills -> ../.agents/skills   symlink so Claude Code sees the skills
.agents/skills/
  implement/                   process for implementing a change (generic)
  code-review/                 review checklists (generic)
  file-organization/           TEMPLATE: default Next.js folder layout
  frontend-design/             TEMPLATE: UI rules + slots for theme and components
  next-best-practices/         Next.js documentation (generalized intro)
  writing-for-agents/          how to write and trim AGENTS.md and skills
  feature-spec/                spec and step-by-step plan for a larger change (edge cases, rollout, progress across sessions)
  pull-request/                user-invoked: push the branch and open a PR with the description template
  grilling/                    stress-testing a plan, round by round
  prototype/                   throwaway prototypes (logic / UI variants)
  improve-codebase-architecture/  finding refactors that deepen modules
  github-issues/               writing and triaging issues
  caveman/                     ultra-concise response mode
```

## Using it in a new project

```bash
NEW=~/projects/<new-project>
git clone https://github.com/marcin6/agent-starter.git /tmp/agent-starter
rsync -a --exclude .git --exclude README.md /tmp/agent-starter/ "$NEW"/
ls -la "$NEW/.claude/skills"   # should be a symlink to ../.agents/skills
```

If your tool doesn't read `AGENTS.md` (plain Claude Code reads `CLAUDE.md`), create a `CLAUDE.md` containing the single line `@AGENTS.md`. Don't add it if `AGENTS.md` is already loaded, or the content will be loaded twice.

## What to fill in (in this order)

1. **`AGENTS.md`**: replace every `<...>` with specifics, delete sections that don't apply (e.g. "Writes and payments" if there are no payments), and finally remove the `<!-- -->` HTML comments. Those comments are loaded into context on every turn.
   - "How the app works": one paragraph per domain area. Write only what the agent can't read from the code: the source of truth, the flow, the reason behind a non-obvious decision, and the "change together" lists.
   - "Risk areas": add project-specific risks. `implement` and `code-review` check this list, so keep it only here.
   - "Code conventions": these are the default conventions (`I`/`T` prefixes, named exports, a single destructured object as the argument, `async`/`await`). Change them before the first line of code, then stick to them.
   - "Checks", "Deployment", "Git": the real script names, hosting and branches.
2. **`file-organization`**: adapt the tables to the project. The backend has placeholders only. Once there is code, put in real examples.
3. **`file-organization` → "Exceptions"**: list deliberate departures from the layout as they appear, or delete the empty section.
4. **`frontend-design`**: fill in "Building blocks" (the components in `src/ui` and their props) and "Look" (font, palette in the Tailwind config, button shape, images). In an empty project, settle the theme first, because the first screens set the pattern for the rest.
5. Delete the skills you don't need. Every model-invoked skill has a description that sits in context all the time.

## Starter principles

- The instructions contain only patterns that carry over between projects: data access with Zod validation, server components by default, amounts computed only on the server, validated `localStorage` state, mobile first, no layout shift, `NEXT_PUBLIC_`/`.env` security rules, a dependency policy (versions older than 2 weeks), and a git workflow (commit/push only on request).
- Domain risks live in one place: `AGENTS.md` → "Risk areas". `implement` and `code-review` refer to that list instead of duplicating it.
- The documentation files in `next-best-practices/*.md` were written for Next.js 15/16, so in a new project the docs of the installed version (`node_modules/next/dist/docs/`) take precedence, as the skill's intro states.

## Maintenance

When you improve a rule in any project and the rule isn't domain-specific, bring it back here so the starter doesn't drift from practice. Use the `writing-for-agents` skill when editing the instructions.

---

<a id="polski"></a>

# Agent starter: instrukcje i skille dla projektu frontend/fullstack

Uniwersalny zestaw instrukcji dla agentów (Claude Code, Codex i inne czytające `AGENTS.md`). Domyślnie pod stack: Next.js App Router + TypeScript + Tailwind + Zod + Vitest, z osobnym backendem (CMS/API) i ewentualnie płatnościami.

Ten README opisuje sam starter. Do nowego projektu go nie kopiuj.

## Co jest w środku

```
AGENTS.md                      szablon głównych instrukcji (do uzupełnienia)
.claude/skills -> ../.agents/skills   symlink, żeby Claude Code widział skille
.agents/skills/
  implement/                   proces wdrażania zmiany (ogólny)
  code-review/                 checklisty review (ogólne)
  file-organization/           SZABLON: domyślny układ katalogów Next.js
  frontend-design/             SZABLON: zasady UI + miejsca na motyw i komponenty
  next-best-practices/         dokumentacja Next.js (wstęp uogólniony)
  writing-for-agents/          jak pisać i przycinać AGENTS.md i skille
  feature-spec/                spec i plan krok po kroku dla większej zmiany (przypadki brzegowe, wdrożenie, postęp między sesjami)
  pull-request/                wywoływany ręcznie: push gałęzi i PR z szablonem opisu
  grilling/                    przepytywanie z planu, runda po rundzie
  prototype/                   prototypy do wyrzucenia (logika / warianty UI)
  improve-codebase-architecture/  szukanie refaktorów pogłębiających moduły
  github-issues/               pisanie i porządkowanie issues
  caveman/                     tryb bardzo zwięzłych odpowiedzi
```

## Jak użyć w nowym projekcie

```bash
NEW=~/projects/<nowy-projekt>
git clone https://github.com/marcin6/agent-starter.git /tmp/agent-starter
rsync -a --exclude .git --exclude README.md /tmp/agent-starter/ "$NEW"/
ls -la "$NEW/.claude/skills"   # powinien być symlink do ../.agents/skills
```

Jeśli Twoje narzędzie nie czyta `AGENTS.md` (zwykły Claude Code czyta `CLAUDE.md`), utwórz `CLAUDE.md` z jedną linią `@AGENTS.md`. Nie dodawaj go, jeśli `AGENTS.md` już się ładuje, bo treść załaduje się dwa razy.

## Co uzupełnić (w tej kolejności)

1. **`AGENTS.md`**: zamień każde `<...>` na konkret, usuń sekcje, które nie dotyczą projektu (np. „Writes and payments”, jeśli nie ma płatności), na końcu usuń komentarze HTML `<!-- -->`. Te komentarze ładują się do kontekstu w każdej turze.
   - „How the app works”: po jednym akapicie na obszar domeny. Wpisuj tylko to, czego agent nie wyczyta z kodu: źródło prawdy, przepływ, powód nieoczywistej decyzji i listy „zmień razem”.
   - „Risk areas”: dopisz ryzyka specyficzne dla projektu. `implement` i `code-review` sprawdzają tę listę, więc trzymaj ją tylko tutaj.
   - „Code conventions”: to domyślne konwencje (prefiksy `I`/`T`, named exports, jeden destrukturyzowany obiekt jako argument, `async`/`await`). Zmień je przed pierwszą linią kodu, potem się ich trzymaj.
   - „Checks”, „Deployment”, „Git”: prawdziwe nazwy skryptów, hosting i gałęzie.
2. **`file-organization`**: dopasuj tabele do projektu. Backend ma same placeholdery. Gdy pojawi się kod, wpisz prawdziwe przykłady.
3. **`file-organization` → „Exceptions”**: wpisuj świadome odstępstwa od układu, gdy się pojawią, albo usuń pustą sekcję.
4. **`frontend-design`**: uzupełnij „Building blocks” (komponenty z `src/ui` i ich propsy) oraz „Look” (font, paleta w configu Tailwinda, kształt przycisków, obrazki). W pustym projekcie najpierw ustal motyw, bo pierwsze ekrany wyznaczają wzorzec dla kolejnych.
5. Usuń skille, których nie potrzebujesz. Każdy model-invoked skill ma opis, który cały czas siedzi w kontekście.

## Założenia startera

- Instrukcje zawierają tylko wzorce, które da się przenieść między projektami: data-access z walidacją Zod, server components domyślnie, kwoty liczone tylko na serwerze, walidacja stanu z `localStorage`, mobile first, brak skaczącego layoutu, zasady bezpieczeństwa `NEXT_PUBLIC_`/`.env`, polityka zależności (wersje starsze niż 2 tygodnie), workflow git (commit/push tylko na prośbę).
- Ryzyka domenowe są w jednym miejscu: `AGENTS.md` → „Risk areas”. `implement` i `code-review` odwołują się do tej listy zamiast ją powielać.
- Pliki dokumentacji w `next-best-practices/*.md` pisano pod Next.js 15/16, więc w nowym projekcie wygrywa dokumentacja zainstalowanej wersji (`node_modules/next/dist/docs/`), co zapisano we wstępie skilla.

## Utrzymanie

Gdy w którymś projekcie poprawisz zasadę, która nie jest związana z domeną, przenieś ją tutaj, żeby starter nie rozjechał się z praktyką. Przy edycji instrukcji użyj skilla `writing-for-agents`.
