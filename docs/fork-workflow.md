# Форк akimovdigital

Правила личной копии Instatic: куда коммитить, куда открывать pull request и как подтягивать автора.

Форк `akimovdigital/Instatic` — личная копия `CoreBunch/Instatic` (MIT). `main` остаётся чистым зеркалом автора, своя работа собирается в `local/instatic-dev`. Устройство совпадает с форком Doop (`akimovdigital/doop`: зеркало `main` и интеграционная ветка `local/doop-dev`). Файлы Doop сюда не копируются.

---

## TL;DR

- `main` — чистое зеркало автора. Обновление только fast-forward с `upstream`. Своих коммитов в `main` нет.
- Рабочая линия — `local/instatic-dev`. Каждая задача — своя ветка `feat/`, `fix/`, `perf/` или `chore/`. Pull request открывается только в `local/instatic-dev`.
- PR задач сливаются squash после зелёных `bun run build`, `bun test`, `bun run lint` и своего ревью diff. Единственное исключение — PR `sync/upstream-<short-sha>` в `local/instatic-dev` сливаются merge-коммитом.
- В `CoreBunch/Instatic` нет pull request, issues, комментариев и пушей без явного «да» Никиты.
- Force push запрещён везде, кроме своей рабочей ветки до открытия pull request.
- Удалять можно только свою уже слитую рабочую ветку.
- Автосинк запускается ежедневно в 06:00 МСК. Синхронизацию также можно выполнить вручную.

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

PR задач сливаются squash через GitHub в `local/instatic-dev`. Единственное исключение — PR `sync/upstream-<short-sha>` в `local/instatic-dev` сливаются merge-коммитом. Squash теряет общую историю с upstream и даёт ложные конфликты при следующих синках.

Перед squash на ветке pull request выполните проверки.

1. `bun install`, если зависимости ещё не стоят.
2. `bun run build`
3. `bun test`
4. `bun run lint`
5. Своё ревью diff: в pull request только файлы задачи, база — `local/instatic-dev`, в `main` нет своих коммитов.

Команды запускаются через Bun. Если проверка красная, pull request остаётся открытым, падение не маскируется и не чинится в обход отчёта.

После успешного squash удаляется только эта слитая рабочая ветка, локально и на `origin`. `main`, `local/instatic-dev`, чужие ветки и ветки автора остаются.

## Синхронизация с автором

Для ручной синхронизации выполните следующие шаги.

1. Выполните `git fetch upstream`.
2. Обновите `main` до `upstream/main` через fast-forward и сделайте обычный push `main` на `origin`.
3. Если `local/instatic-dev` уже содержит обновлённый `main`, синк завершён. Иначе создайте от `main` ветку `sync/upstream-<sha>`. Возьмите первые семь символов SHA, например `sync/upstream-967b438`.
4. Откройте PR в `local/instatic-dev`. После проверки результата слияния слейте PR merge-коммитом.

## Автосинк (GitHub Actions)

`.github/workflows/upstream-sync.yml` запускается ежедневно в 03:00 UTC (06:00 МСК). Ручной запуск — `gh workflow run upstream-sync.yml -R akimovdigital/Instatic --ref local/instatic-dev`.

1. Обновляет `main` через GitHub API `merge-upstream` только fast-forward. Если `main` разошёлся с upstream, запуск красный. Force push запрещён.
2. Если `local/instatic-dev` уже содержит upstream HEAD, запуск зелёный с итогом «Форк синхронен».
3. Иначе создаёт ветку `sync/upstream-<short-sha>` и открывает PR. Старые открытые sync PR закрываются как заменённые, их ветки сохраняются.
4. Проверяет конфликты и запускает build/lint/test через `fork-ci.yml` на результате слияния. При конфликте PR остаётся открытым с меткой `sync-conflict`. При падении проверок PR остаётся открытым с комментарием и ссылкой на запуск.
5. После зелёных проверок сливает PR merge-коммитом и удаляет слитую sync-ветку. Если слияние заблокировано, PR получает метку `needs-manual-merge`.

У `GITHUB_TOKEN` нет права менять `.github/workflows/*`. Если upstream меняет workflow-файлы и GitHub отклоняет обновление, запуск завершается с подсказкой. Обновите зеркало вручную через `gh` с правом `workflow` или добавьте секрет `SYNC_TOKEN` с fine-grained PAT. Нужны права read/write для Contents, Pull requests и Workflows на `akimovdigital/Instatic`. При наличии секрета workflow использует его автоматически.

Ветка по умолчанию — `local/instatic-dev`, так как расписание работает только с ветки по умолчанию. В настройках Actions включено «Allow GitHub Actions to create and approve pull requests».

`fork-ci.yml` также запускает build/lint/test на каждом PR в `local/instatic-dev`. Upstream `ci.yml` проверяет `main`.

## Upstream

В `https://github.com/CoreBunch/Instatic` нет pull request, issues, комментариев и пушей, пока Никита явно не скажет «да». Remote `upstream` используется для fetch и fast-forward.

## Force push

Force push запрещён на `main`, на `local/instatic-dev`, на `upstream` и на любую ветку, у которой уже открыт pull request. Единственное исключение — своя рабочая ветка задачи до открытия её pull request.

## Related

- `CLAUDE.md` — раздел «Форк akimovdigital». `AGENTS.md` — symlink на этот файл.
- `.cursor/rules/fork-workflow.mdc` — те же правила с `alwaysApply: true`.
- `.github/workflows/upstream-sync.yml` — расписание и шаги автосинка.
- `.github/workflows/fork-ci.yml` — проверки PR и результата слияния.
- `https://github.com/akimovdigital/doop` — та же схема форка: `main` и `local/doop-dev`.
- `https://github.com/CoreBunch/Instatic` — upstream. Чтение и fast-forward `main`.
