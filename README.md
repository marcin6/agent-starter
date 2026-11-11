# Agent starter: instructions and skills for web and mobile projects

[Wersja polska poniżej](#polski)

A universal set of instructions for coding agents (Claude Code, Codex and any other tool that reads `AGENTS.md`), in two variants:

- **web**: Next.js App Router + TypeScript + Tailwind + Zod + Vitest, with a separate backend (CMS/API) and optional payments;
- **mobile**: Expo (development builds, Continuous Native Generation) + Expo Router + TypeScript + TanStack Query + Zod + Jest / React Native Testing Library + Maestro, built and released with EAS, with a separate backend (API) and optional purchases.

A project gets `core/` plus one variant. This README describes the starter itself. Don't copy it into a new project.

## What's inside

```
core/                          identical for every project
  .claude/skills -> ../.agents/skills   symlink so Claude Code sees the skills
  .agents/skills/
    writing-for-agents/        how to write and trim AGENTS.md and skills
    grilling/                  stress-testing a plan, round by round
    improve-codebase-architecture/  finding refactors that deepen modules
    github-issues/             writing and triaging issues
    caveman/                   ultra-concise response mode

web/                           Next.js variant
  AGENTS.md                    main instructions template (to fill in)
  .agents/skills/
    implement/                 process for implementing a change
    code-review/               review checklists
    feature-spec/              spec and step-by-step plan for a larger change
    pull-request/              user-invoked: push the branch and open a PR with the description template
    file-organization/         TEMPLATE: default Next.js folder layout
    frontend-design/           TEMPLATE: UI rules + slots for theme and components
    next-best-practices/       Next.js documentation (generalized intro)
    prototype/                 throwaway prototypes (logic / UI variants on a page)

mobile/                        Expo / React Native variant
  AGENTS.md                    main instructions template (to fill in)
  .agents/skills/
    implement/                 process for implementing a change (JS-only vs native, both platforms)
    code-review/               review checklists
    feature-spec/              spec and step-by-step plan for a larger change
    pull-request/              user-invoked: push the branch and open a PR with the description template
    file-organization/         TEMPLATE: default Expo Router folder layout
    mobile-design/             TEMPLATE: UI rules for iOS/Android + slots for theme and components
    expo-best-practices/       Expo / React Native guides, one file per area:
      routing.md                 Expo Router, params, deep links, protected routes
      data-fetching.md           API client, TanStack Query, foreground/offline refetching
      performance.md             lists, images, UI thread, startup, measuring in release builds
      native-config.md           dev builds, CNG, app.config, env variables, permissions
      storage-security.md        SecureStore / MMKV / SQLite, versioned storage, what is public
      debugging.md               common errors, logs and tools per platform
    mobile-testing/            Jest + React Native Testing Library + Maestro
    release/                   JS-only vs native, versions, runtime version, OTA updates, store submission
    prototype/                 throwaway prototypes (logic / UI variants on a screen)
```

Skills with the same name in `web/` and `mobile/` (`implement`, `code-review`, `feature-spec`, `pull-request`, `file-organization`, `prototype`) share their structure and differ where the platform differs. Skills that are identical for both live only in `core/`.

### How the variants differ

| Web | Mobile |
| --- | --- |
| `frontend-design` | `mobile-design`: safe areas, keyboard, touch targets, platform conventions, font scaling, VoiceOver/TalkBack, dark mode |
| `next-best-practices` | `expo-best-practices`: Expo Router, TanStack Query with app state and network, lists, native config, storage, debugging |
| Tests in `implement` and `AGENTS.md` | `mobile-testing`: Jest, React Native Testing Library, Maestro |
| Deployment = merge to the release branch | `release`: every change is JS-only (OTA update) or native (new build + store review); old versions stay installed |
| Risk areas: money, persisted state, SEO, server/client boundary | Risk areas: native vs JS, old app versions, money and in-app purchases, persisted state, secrets in the bundle, permissions and privacy, deep links, platform parity |

## Using it in a new project

```bash
NEW=~/projects/<new-project>
VARIANT=web   # or: mobile
git clone https://github.com/marcin6/agent-starter.git /tmp/agent-starter
rsync -a /tmp/agent-starter/core/ "$NEW"/
rsync -a /tmp/agent-starter/$VARIANT/ "$NEW"/
ls -la "$NEW/.claude/skills"   # should be a symlink to ../.agents/skills
ls "$NEW/.agents/skills"       # core skills + the variant's skills
```

For mobile, create the app first (`npx create-expo-app@latest`) and copy the starter into it, or copy the starter into an empty folder and create the app inside it. The Expo template already ignores `ios/` and `android/`, as Continuous Native Generation expects.

If your tool doesn't read `AGENTS.md` (plain Claude Code reads `CLAUDE.md`), create a `CLAUDE.md` containing the single line `@AGENTS.md`. Don't add it if `AGENTS.md` is already loaded, or the content will be loaded twice.

## What to fill in (in this order)

1. **`AGENTS.md`**: replace every `<...>` with specifics, delete sections that don't apply (e.g. "Writes and payments" / "Purchases and payments" if nothing is sold), and finally remove the `<!-- -->` HTML comments. Those comments are loaded into context on every turn.
   - "How the app works": one paragraph per domain area. Write only what the agent can't read from the code: the source of truth, the flow, the reason behind a non-obvious decision, and the "change together" lists.
   - "Risk areas": add project-specific risks. `implement` and `code-review` check this list, so keep it only here.
   - "Code conventions": these are the default conventions (`I`/`T` prefixes, named exports, a single destructured object as the argument, `async`/`await`). Change them before the first line of code, then stick to them.
   - "Checks", "Deployment", "Git": the real script names, hosting (EAS profiles and channels for mobile) and branches.
2. **`file-organization`**: adapt the tables to the project. The backend has placeholders only. Once there is code, put in real examples. List deliberate departures from the layout under "Exceptions" as they appear, or delete the empty section.
3. **`frontend-design`** (web) or **`mobile-design`** (mobile): fill in "Building blocks" (the components in `src/ui` and their props) and "Look" (font, palette, button shape; for mobile also light and dark color tokens, spacing scale, keyboard handling). In an empty project, settle the theme first, because the first screens set the pattern for the rest.
4. Mobile only: **`release`** (how build numbers are managed, the runtime version policy, how the app enforces a minimum version), then the few `<...>` slots in **`expo-best-practices`** and **`mobile-testing`** (list component, storage library, test helper, error tracker).
5. Delete the skills you don't need. Every model-invoked skill has a description that sits in context all the time.

## Starter principles

- The instructions contain only patterns that carry over between projects.
  - Both variants: data access with Zod validation, validated persisted state, amounts computed only on the server, secrets kept out of anything shipped to the client, a dependency policy (versions older than 2 weeks), and a git workflow (commit/push only on request).
  - Web: server components by default, mobile first, no layout shift, `NEXT_PUBLIC_`/`.env` rules.
  - Mobile: TanStack Query wired to app state and network, every screen openable from a deep link, tokens only in SecureStore, `EXPO_PUBLIC_` treated as public, permissions requested in context, both platforms checked, every change classified as JS-only or native, `npx expo install` for SDK-compatible versions.
- Domain risks live in one place: `AGENTS.md` → "Risk areas". `implement` and `code-review` refer to that list instead of duplicating it.
- Framework guides defer to the installed version. The files in `web/.agents/skills/next-best-practices/*.md` were written for Next.js 15/16, so the docs of the installed version (`node_modules/next/dist/docs/`) take precedence. The guides in `mobile/.agents/skills/expo-best-practices/` name no SDK version; the documentation for the installed SDK takes precedence.

## Maintenance

When you improve a rule in any project and the rule isn't domain-specific, bring it back here so the starter doesn't drift from practice:

- a change to a skill in `core/` applies to every project as is;
- a change to a skill that exists in both `web/` and `mobile/`: apply it to both in the same commit, adapted to the platform, unless it only concerns one platform;
- a skill that becomes identical in both variants moves to `core/`.

Use the `writing-for-agents` skill when editing the instructions.

---

<a id="polski"></a>

# Agent starter: instrukcje i skille dla projektów web i mobile

Uniwersalny zestaw instrukcji dla agentów (Claude Code, Codex i inne czytające `AGENTS.md`) w dwóch wariantach:

- **web**: Next.js App Router + TypeScript + Tailwind + Zod + Vitest, z osobnym backendem (CMS/API) i ewentualnie płatnościami;
- **mobile**: Expo (development buildy, Continuous Native Generation) + Expo Router + TypeScript + TanStack Query + Zod + Jest / React Native Testing Library + Maestro, budowanie i wydawanie przez EAS, z osobnym backendem (API) i ewentualnie zakupami.

Projekt dostaje `core/` i jeden wariant. Ten README opisuje sam starter. Do nowego projektu go nie kopiuj.

## Co jest w środku

```
core/                          identyczne dla każdego projektu
  .claude/skills -> ../.agents/skills   symlink, żeby Claude Code widział skille
  .agents/skills/
    writing-for-agents/        jak pisać i przycinać AGENTS.md i skille
    grilling/                  przepytywanie z planu, runda po rundzie
    improve-codebase-architecture/  szukanie refaktorów pogłębiających moduły
    github-issues/             pisanie i porządkowanie issues
    caveman/                   tryb bardzo zwięzłych odpowiedzi

web/                           wariant Next.js
  AGENTS.md                    szablon głównych instrukcji (do uzupełnienia)
  .agents/skills/
    implement/                 proces wdrażania zmiany
    code-review/               checklisty review
    feature-spec/              spec i plan krok po kroku dla większej zmiany
    pull-request/              wywoływany ręcznie: push gałęzi i PR z szablonem opisu
    file-organization/         SZABLON: domyślny układ katalogów Next.js
    frontend-design/           SZABLON: zasady UI + miejsca na motyw i komponenty
    next-best-practices/       dokumentacja Next.js (wstęp uogólniony)
    prototype/                 prototypy do wyrzucenia (logika / warianty UI na stronie)

mobile/                        wariant Expo / React Native
  AGENTS.md                    szablon głównych instrukcji (do uzupełnienia)
  .agents/skills/
    implement/                 proces wdrażania zmiany (JS vs natywne, obie platformy)
    code-review/               checklisty review
    feature-spec/              spec i plan krok po kroku dla większej zmiany
    pull-request/              wywoływany ręcznie: push gałęzi i PR z szablonem opisu
    file-organization/         SZABLON: domyślny układ katalogów Expo Router
    mobile-design/             SZABLON: zasady UI dla iOS/Androida + miejsca na motyw i komponenty
    expo-best-practices/       przewodniki Expo / React Native, jeden plik na obszar:
      routing.md                 Expo Router, parametry, deep linki, chronione trasy
      data-fetching.md           klient API, TanStack Query, odświeżanie po powrocie do aplikacji i offline
      performance.md             listy, obrazki, wątek UI, start aplikacji, pomiar w buildzie release
      native-config.md           development buildy, CNG, app.config, zmienne środowiskowe, uprawnienia
      storage-security.md        SecureStore / MMKV / SQLite, wersjonowany storage, co jest publiczne
      debugging.md               typowe błędy, logi i narzędzia na każdej platformie
    mobile-testing/            Jest + React Native Testing Library + Maestro
    release/                   JS vs natywne, wersje, runtime version, aktualizacje OTA, wysyłka do sklepów
    prototype/                 prototypy do wyrzucenia (logika / warianty UI na ekranie)
```

Skille o tej samej nazwie w `web/` i `mobile/` (`implement`, `code-review`, `feature-spec`, `pull-request`, `file-organization`, `prototype`) mają wspólną strukturę i różnią się tam, gdzie różni się platforma. Skille identyczne dla obu są tylko w `core/`.

### Czym różnią się warianty

| Web | Mobile |
| --- | --- |
| `frontend-design` | `mobile-design`: safe area, klawiatura, wielkość pól dotyku, konwencje platform, skalowanie czcionki, VoiceOver/TalkBack, tryb ciemny |
| `next-best-practices` | `expo-best-practices`: Expo Router, TanStack Query ze stanem aplikacji i sieci, listy, konfiguracja natywna, storage, debugowanie |
| Testy w `implement` i `AGENTS.md` | `mobile-testing`: Jest, React Native Testing Library, Maestro |
| Wdrożenie = merge do gałęzi release | `release`: każda zmiana to albo tylko JS (aktualizacja OTA), albo natywna (nowy build + review w sklepie); stare wersje zostają na telefonach |
| Ryzyka: pieniądze, stan w przeglądarce, SEO, granica serwer/klient | Ryzyka: natywne vs JS, stare wersje aplikacji, pieniądze i zakupy w aplikacji, stan na urządzeniu, sekrety w bundlu, uprawnienia i prywatność, deep linki, zgodność obu platform |

## Jak użyć w nowym projekcie

```bash
NEW=~/projects/<nowy-projekt>
VARIANT=web   # albo: mobile
git clone https://github.com/marcin6/agent-starter.git /tmp/agent-starter
rsync -a /tmp/agent-starter/core/ "$NEW"/
rsync -a /tmp/agent-starter/$VARIANT/ "$NEW"/
ls -la "$NEW/.claude/skills"   # powinien być symlink do ../.agents/skills
ls "$NEW/.agents/skills"       # skille z core + skille wariantu
```

Dla mobile najpierw utwórz aplikację (`npx create-expo-app@latest`) i skopiuj do niej starter, albo skopiuj starter do pustego folderu i utwórz w nim aplikację. Szablon Expo sam ignoruje `ios/` i `android/`, czego wymaga Continuous Native Generation.

Jeśli Twoje narzędzie nie czyta `AGENTS.md` (zwykły Claude Code czyta `CLAUDE.md`), utwórz `CLAUDE.md` z jedną linią `@AGENTS.md`. Nie dodawaj go, jeśli `AGENTS.md` już się ładuje, bo treść załaduje się dwa razy.

## Co uzupełnić (w tej kolejności)

1. **`AGENTS.md`**: zamień każde `<...>` na konkret, usuń sekcje, które nie dotyczą projektu (np. „Writes and payments” / „Purchases and payments”, jeśli nic nie sprzedajesz), na końcu usuń komentarze HTML `<!-- -->`. Te komentarze ładują się do kontekstu w każdej turze.
   - „How the app works”: po jednym akapicie na obszar domeny. Wpisuj tylko to, czego agent nie wyczyta z kodu: źródło prawdy, przepływ, powód nieoczywistej decyzji i listy „zmień razem”.
   - „Risk areas”: dopisz ryzyka specyficzne dla projektu. `implement` i `code-review` sprawdzają tę listę, więc trzymaj ją tylko tutaj.
   - „Code conventions”: to domyślne konwencje (prefiksy `I`/`T`, named exports, jeden destrukturyzowany obiekt jako argument, `async`/`await`). Zmień je przed pierwszą linią kodu, potem się ich trzymaj.
   - „Checks”, „Deployment”, „Git”: prawdziwe nazwy skryptów, hosting (dla mobile profile i kanały EAS) i gałęzie.
2. **`file-organization`**: dopasuj tabele do projektu. Backend ma same placeholdery. Gdy pojawi się kod, wpisz prawdziwe przykłady. Świadome odstępstwa od układu wpisuj w „Exceptions”, gdy się pojawią, albo usuń pustą sekcję.
3. **`frontend-design`** (web) albo **`mobile-design`** (mobile): uzupełnij „Building blocks” (komponenty z `src/ui` i ich propsy) oraz „Look” (font, paleta, kształt przycisków; dla mobile także tokeny kolorów dla jasnego i ciemnego motywu, skala odstępów, obsługa klawiatury). W pustym projekcie najpierw ustal motyw, bo pierwsze ekrany wyznaczają wzorzec dla kolejnych.
4. Tylko mobile: **`release`** (jak są zarządzane numery buildów, polityka runtime version, sposób wymuszania minimalnej wersji), potem kilka miejsc `<...>` w **`expo-best-practices`** i **`mobile-testing`** (komponent listy, biblioteka storage, helper testowy, narzędzie do zbierania błędów).
5. Usuń skille, których nie potrzebujesz. Każdy model-invoked skill ma opis, który cały czas siedzi w kontekście.

## Założenia startera

- Instrukcje zawierają tylko wzorce, które da się przenieść między projektami.
  - Oba warianty: data-access z walidacją Zod, walidacja zapisanego stanu, kwoty liczone tylko na serwerze, sekrety poza wszystkim, co trafia do klienta, polityka zależności (wersje starsze niż 2 tygodnie), workflow git (commit/push tylko na prośbę).
  - Web: server components domyślnie, mobile first, brak skaczącego layoutu, zasady `NEXT_PUBLIC_`/`.env`.
  - Mobile: TanStack Query podłączone do stanu aplikacji i sieci, każdy ekran do otwarcia z deep linka, tokeny tylko w SecureStore, `EXPO_PUBLIC_` traktowane jako publiczne, uprawnienia systemowe proszone w kontekście, sprawdzanie obu platform, klasyfikacja każdej zmiany jako JS albo natywnej, `npx expo install` dla wersji zgodnych z SDK.
- Ryzyka domenowe są w jednym miejscu: `AGENTS.md` → „Risk areas”. `implement` i `code-review` odwołują się do tej listy zamiast ją powielać.
- Przewodniki frameworków ustępują zainstalowanej wersji. Pliki w `web/.agents/skills/next-best-practices/*.md` pisano pod Next.js 15/16, więc wygrywa dokumentacja zainstalowanej wersji (`node_modules/next/dist/docs/`). Przewodniki w `mobile/.agents/skills/expo-best-practices/` nie podają wersji SDK; wygrywa dokumentacja zainstalowanego SDK.

## Utrzymanie

Gdy w którymś projekcie poprawisz zasadę, która nie jest związana z domeną, przenieś ją tutaj, żeby starter nie rozjechał się z praktyką:

- zmiana skilla w `core/` dotyczy każdego projektu bez zmian;
- zmianę skilla, który istnieje w `web/` i `mobile/`, wprowadź w obu w tym samym commicie, dopasowaną do platformy, chyba że dotyczy tylko jednej;
- skill, który w obu wariantach stał się identyczny, przenieś do `core/`.

Przy edycji instrukcji użyj skilla `writing-for-agents`.
