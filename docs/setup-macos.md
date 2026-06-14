# Установка на чистый macOS

Гайд по разворачиванию `mykindle` на свежем Mac (Apple Silicon или Intel).
Нужны права администратора. Все шаги — через [Homebrew](https://brew.sh).

> Зачем именно так: на современных прошивках Kindle сайдлоад **AZW3 даёт чёрный
> экран** (формат KF8 не рендерится с прошивки 5.19.2+). Рабочие форматы для
> заливки по USB — **MOBI6** (простое форматирование, ставится только Calibre) и
> **KFX** (богатое форматирование, требует Kindle Previewer). Поэтому ниже ставим
> и то, и другое.

## 1. Homebrew (если ещё нет)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 2. Rosetta 2 (только Apple Silicon — M1/M2/M3/M4/M5)

Бинарь заливки `mtp-cli` — x86_64, ему нужен Rosetta:

```bash
softwareupdate --install-rosetta --agree-to-license
```

## 3. Calibre — конвертер и CLI

```bash
brew install --cask calibre
```
CLI-инструменты появятся в `/Applications/calibre.app/Contents/MacOS/`
(`ebook-convert`, `ebook-meta`, `calibredb`, `calibre-customize`, …).

## 4. OpenMTP — содержит `mtp-cli` для заливки на Kindle

```bash
brew install --cask openmtp
```
Заливочный CLI лежит внутри бандла:
`/Applications/OpenMTP.app/Contents/Resources/bin/mtp-cli`
(заявляет USB напрямую, GUI для заливки не нужен).

## 5. Kindle Previewer — движок для KFX (богатое форматирование)

```bash
brew install --cask kindle-previewer
```
⚠️ **Первый запуск — вручную из GUI:** открой «Kindle Previewer 3» и прими
лицензионное соглашение. Без этого конвертация в KFX из CLI не пойдёт. Это
делается один раз.

## 6. Плагины KFX для Calibre

Скачиваются с официального индекса плагинов Calibre и ставятся в CLI:

```bash
CAL="/Applications/calibre.app/Contents/MacOS"
cd /tmp
curl -fsSL -o kfx-output.zip https://plugins.calibre-ebook.com/272407.zip   # KFX Output
curl -fsSL -o kfx-input.zip  https://plugins.calibre-ebook.com/291290.zip   # KFX Input
"$CAL/calibre-customize" -a /tmp/kfx-output.zip
"$CAL/calibre-customize" -a /tmp/kfx-input.zip
```
После этого `ebook-convert in.epub out.kfx` начинает работать (плагин дёргает
Kindle Previewer для сборки KFX).

## 7. Клонировать репозиторий

```bash
git clone https://github.com/kscht/mykindle.git ~/mykindle
cd ~/mykindle
```
Обновляться потом — `git pull`.

Сам инструмент `bin/kindle` — на **Python 3** (без сторонних зависимостей).
На macOS python3 обычно уже есть (Command Line Tools); если нет — `xcode-select
--install` или `brew install python`.

## 8. Первый запуск

```bash
cd ~/mykindle
./bin/kindle status            # проверка: inbox / staging / Kindle
./bin/kindle convert --jobs 6  # inbox → staging (KFX, параллельно)
./bin/kindle push              # staging → Kindle (воткни устройство)
```
Подробно про команды и теги — в [`../README.md`](../README.md).

## Пути к инструментам (PATH в неинтерактивной сессии их не подхватывает)

| Инструмент | Путь |
|---|---|
| Calibre CLI | `/Applications/calibre.app/Contents/MacOS/` |
| Заливка MTP | `/Applications/OpenMTP.app/Contents/Resources/bin/mtp-cli` |
| Homebrew | `/opt/homebrew/bin/brew` (Apple Silicon) |

## Проверка установки

```bash
CAL="/Applications/calibre.app/Contents/MacOS"
"$CAL/ebook-convert" --version
"$CAL/calibre-customize" -l | grep -i kfx                 # должны быть KFX Output / KFX Input
ls -d "/Applications/Kindle Previewer 3.app"
/Applications/OpenMTP.app/Contents/Resources/bin/mtp-cli --version
```

## Форматы вывода (кратко)

| Формат | Богатое форматирование | Сайдлоад на новых Kindle |
|---|---|---|
| **KFX** | ✅ да (enhanced typesetting) | ✅ работает |
| **MOBI6** | ⚠️ простое | ✅ работает (совместим со всеми) |
| AZW3 (KF8) | ✅ да | ❌ чёрный экран на 5.19.2+ |
| EPUB | — | ❌ сайдлоадом не распознаётся |

Подробнее о выборе формата — `../CLAUDE.md`.
