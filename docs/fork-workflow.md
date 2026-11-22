# Форк akimovdigital

Правила личной копии Instatic: куда коммитить, куда открывать pull request и как подтягивать автора.

Форк `akimovdigital/Instatic` — личная копия `CoreBunch/Instatic` (MIT). `main` остаётся чистым зеркалом автора, своя работа собирается в `local/instatic-dev`. Устройство совпадает с форком Doop (`akimovdigital/doop`: зеркало `main` и интеграционная ветка `local/doop-dev`). Файлы Doop сюда не копируются.

---

## TL;DR

- `main` — чистое зеркало автора. Обновление только fast-forward с `upstream`. Своих коммитов в `main` нет.
- Рабочая линия — `local/instatic-dev`. Каждая задача — своя ветка `feat/`, `fix/`, `perf/` или `chore/`. Pull request открывается только в `local/instatic-dev`.
- Squash в `local/instatic-dev` только после зелёных `bun run build`, `bun test`, `bun run lint` и своего ревью diff.
- В `CoreBunch/Instatic` нет pull request, issues, комментариев и пушей без явного «да» Никиты.
- Force push запрещён везде, кроме своей рабочей ветки до открытия pull request.
- Удалять можно только свою уже слитую рабочую ветку.
- Перед каждой задачей и не реже раза в неделю: fast-forward `main`, затем ветка `sync/upstream-<sha>` и pull request в `local/instatic-dev`.

## Ветки

### `main`

`origin` — `https://github.com/akimovdigital/Instatic`. `upstream` — `https://github.com/CoreBunch/Instatic.git`.

`main` на `origin` совпадает с `upstream/main`. Обновление: `git fetch upstream`, затем `git merge --ff-only` (или другой fast-forward) в `main` и обычный `git push origin main`. Если fast-forward невозможен, работа останавливается: мешающие коммиты и SHA обеих сторон фиксируются в отчёте, свои коммиты в `main` не появляются, история не переписывается.

Прямой push в `main` бывает только таким обновлением зеркала.

### `local/instatic-dev`

Сюда сливается вся своя работа. Ветка начинается от обновлённого `main`. Дальше она живёт отдельно: коммиты автора приходят pull request-ом синхронизации, а не коммитом в `main`.

### Ветка задачи

Ветка задачи создаётся от `local/instatic-dev`:

- `feat/<краткое-описание>`
- `fix/<краткое-описание>`
- `perf/<краткое-описание>`
- `chore/<краткое-описание>`

Один pull request — одна задача. Заголовок — Conventional Commits: `<type>(<scope>): <summary>`.

## Слияние

Squash через GitHub в `local/instatic-dev`. Merge-коммит не используется. Перед squash на ветке pull request:

1. `bun install`, если зависимости ещё не стоят.
2. `bun run build`
3. `bun test`
4. `bun run lint`
5. Своё ревью diff: в pull request только файлы задачи, база — `local/instatic-dev`, в `main` нет своих коммитов.

Команды запускаются через Bun. Если проверка красная, pull request остаётся открытым, падение не маскируется и не чинится в обход отчёта.

После успешного squash удаляется только эта слитая рабочая ветка, локально и на `origin`. `main`, `local/instatic-dev`, чужие ветки и ветки автора остаются.

## Синхронизация с автором

Перед каждой задачей и не реже одного раза в неделю:

1. `git fetch upstream`.
2. Fast-forward `main` до `upstream/main` и обычный push `main` на `origin`.
3. От обновлённого `main` создаётся ветка `sync/upstream-<sha>`. `<sha>` — полный SHA этого `main`.
4. Pull request ветки `sync/upstream-<sha>` открывается в `local/instatic-dev`.

## Upstream

В `https://github.com/CoreBunch/Instatic` нет pull request, issues, комментариев и пушей, пока Никита явно не скажет «да». Remote `upstream` используется для fetch и fast-forward.

## Force push

Force push запрещён на `main`, на `local/instatic-dev`, на `upstream` и на любую ветку, у которой уже открыт pull request. Единственное исключение — своя рабочая ветка задачи до открытия её pull request.

## Related

- `CLAUDE.md` — раздел «Форк akimovdigital». `AGENTS.md` — symlink на этот файл.
- `.cursor/rules/fork-workflow.mdc` — те же правила с `alwaysApply: true`.
- `https://github.com/akimovdigital/doop` — та же схема форка: `main` и `local/doop-dev`.
- `https://github.com/CoreBunch/Instatic` — upstream. Чтение и fast-forward `main`.
