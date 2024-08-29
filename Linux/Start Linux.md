Тут я пропишу некотрые штуки для установки лтнукса.

---
## 1.  Основа основ
Для начала нужна флешка для линукса.  Скачиваем ==тут будет ссылка== и с помощью [Rufus](https://rufus.ie/ru/) делаем установочную флешку.
###### Покажу когда буду с винды ИУИУИУИУ
## 2. Установка
Пропишу немного  про базу и что выбирал я при установке. (Будет позже, сейчас лень)
Для обноваления пакетиков используем: 
```shell
yay -Suy
```
или 
```shell
sudo pacman -Suy
```
### 1. Выбор *Менеджера окон*
При запуске пк, мы заходим в Линукс без рабочего стола и прописываем команды для установки. Я использую **[hyprdots](https://github.com/prasanthrangan/hyprdots)** . Ниже дублировал команды. 
- Установка:
```bash
git clone --depth 1 https://github.com/prasanthrangan/hyprdots ~/HyDE
cd ~/HyDE/Scripts
./install.sh
```
- Обновление:
```bash
cd ~/HyDE/Scripts
git pull
./install.sh -r
```
## 3.  Настройка Линукса
Такс, самое мое *нелюбимое*, это настройка линукса. При запуске пк и входе в систему, мы должны настроить герцовку, разрешение и язык. 
### 1. Монитор 
Используя [[hotkey linux]] мы открываем VScode и открываем все файлы. В них мы заходим по пути `.config/hypr/monitors.conf` и `.config/hypr/hyprland.conf`  в них мы прописываем ваш монитор и разрешение, у меня так:
```Properties
monitor = name, resolution, position, scale

monitor = ,3440x1440@144,auto,auto
```
### 2. Смена языка 
Добавление языка происходит в файле `.config/hypr/hyprland.conf`. В нем нужно найти раздел Input и добавить язык через запятую:
```Properties
input {
	kb_layout = us, ru // ДОБАВЛЯЕМ ТУТА
	follow_mouse = 1

	touchpad {
		natural_scroll = no
	}
	
	sensitivity = 0
	force_no_accel = 1
}
```
Готово, теперь есть плавность и русский язык, ура!

> [!Другие файлы]
> Немного пропишу еще настройки, может быть пригодиться.
> - *keybindings.conf* - используется для настройки [[hotkey linux]]
> - *animations.conf* - используется для настройки анимаций
> - *windowrules.conf* - нужен для настройки прозрачности приложений
## 4.  Установка приложений
Ну что же вот и конец, я просто пропишу какие приложения ставил Я. Открываем терминал (kitty) через [[hotkey linux]] и прописываем команды и совсем соглашаемся.
- VScode - его сначала нужно удалить и поставить **нормальный**!
```shell
yay -R code
yay -S visual-studio-code-bin
```
- Telegram - нужно же как-то общаться :)
```shell
yay -S telegram-desktop
```
- Spotify - навалим музла
```shell
yay -S spotify-launcher
```
- Nodejs - нодааааааааааа
```shell
yay -S nodejs
```
- Ni 
```shell
bun i -g @antfu/ni
```
- Bun
```shell
curl -fsSL https://bun.sh/install | bash
```
- Obsidian - нодааааааааааа
```shell
yay -S obsidian
```
Пока что усе!