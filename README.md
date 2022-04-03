# MarkBot
## bot-assistant for steam
---
## travel guide:
* [Import block](#Import-Block)
* [Recognize Text](#Recognize-Text)
* [Open Steam Account](#Open-Steam-Account)
* [Trade Offer](#Trade-Offer)
* [Sale Of Items](#Sale-Of-Items)
* [Return Items](#Return-Items)
* [Editting Profile](#Editting-Profile)
* [Delete The Game](#Delete-The-Game)
* [Work With Trade Platform](#Work-With-Trade-Platform)
* [Launch Bot](#Launch-Bot)
* [Greeting](#Greeting)
* [Command Reading Cycle](#Command-Reading-Cycle)
---
### Import Block [Go To TOP](#TOP)
```py
from random import choice
import speech_recognition as sr
from selenium import webdriver
from time import sleep
from data import *
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys

# options = webdriver.ChromeOptions()
# options.headless = True
# driver = webdriver.Chrome(options=options)
driver = webdriver.Chrome()
recognize_txt = '_'
g = 1
extreme_price = 40
wait_for_conf = {}
```
### Recognize text
```py
# Функция распознавания речи
def rec_txt():
    global recognize_txt
    recognize_txt = ''
    while recognize_txt == '':
        recognizer = sr.Recognizer()
        with sr.Microphone() as source:
            audio = recognizer.listen(source)
        try:
            recognize_txt = str(recognizer.recognize_google(audio, language='ru'))
        except:
            continue
    print(recognize_txt)
    # recognize_txt = input()
```
### Open Steam Account
```py
# Функция входа в аккаунт Steam
def confirm_to_steam():
    global driver
    driver.get('https://store.steampowered.com/')
    driver.find_element(By.XPATH, '//*[@id="global_action_menu"]/a').click()
    driver.find_element(By.XPATH, '//*[@id="input_username"]').send_keys(login)
    driver.find_element(By.XPATH, '//*[@id="input_password"]').send_keys(password)
    driver.find_element(By.XPATH, '//*[@id="login_btn_signin"]/button/span').click()
    print(f'Send your Guard Code: ', end='')
    driver.find_element(By.XPATH, '//*[@id="twofactorcode_entry"]').send_keys(input())
    driver.find_element(By.XPATH, '//*[@id="login_twofactorauth_buttonset_entercode"]/div[1]').click()
    sleep(2)
```
### Trade Offer 
```py
#  Класс отправки предложения обмена
class TradeOffer:
    def __init__(self):
        global driver

    def enter_trade_menu(self):
        driver.find_element(By.XPATH, '//*[@id="global_header"]/div/div[2]/a[3]').click()
        sleep(0.5)
        driver.find_element(By.XPATH, '//*[@id="friendactivity_right_column"]/div/div[3]/div[8]/a/span').click()

    def create_new_trade_offer(self):
        self.enter_trade_menu()
        sleep(1)
        driver.find_element(By.XPATH,
                            '/html/body/div[1]/div[7]/div[2]/div/div[2]/div[2]/div/div[1]/div[1]/div/span').click()
        print('Choose the recipient.')
```
### Sale Of Items
```py
#  Класс продажи вещей
class SaleOfItems:
    def __init__(self):
        global driver

    # функция входа в инвентарь
    def enter_inventory(self):
        driver.find_element(By.XPATH, '//*[@id="global_header"]/div/div[2]/a[3]').click()
        sleep(0.5)
        driver.find_element(By.XPATH, '//*[@id="friendactivity_right_column"]/div/div[3]/div[7]/a/span').click()
        sleep(0.5)
        driver.find_element(By.XPATH, f'//*[text()="{game_name}"]').click()

    # функция определения имени предмета
    def item_name(self, g):
        return driver.find_element(By.XPATH, f'//*[@id="iteminfo{g}_item_name"]').text.lower()

    # функция проверки на возможность товара быть проданным
    def may_or_not(self):
        if ('Нельзя продать' in driver.find_element(By.ID, f'iteminfo0_item_tags_content').text) or (
                'Нельзя продать' in driver.find_element(By.ID, f'iteminfo1_item_tags_content').text) or (
                driver.find_element(By.XPATH, '//*[@id="empty_filtered_inventory_page"]/div').is_displayed()):
            return 1
        return 0

    def is_last(self, i):
        if (i + 1) % 25 == 0 and i != 0:
            driver.find_element(By.ID, "pagebtn_next").click()
            sleep(2)
        if i == self.count_of_items() - 1:
            print('There are no more items with this type in your inventory!')
            return 33

    # функция получения цены предмета
    def get_price_of_item(self):
        global g
        if driver.find_element(By.XPATH, '//*[@id="iteminfo1_item_market_actions"]/a/span[2]').is_displayed():
            g = 1
        if driver.find_element(By.XPATH, '//*[@id="iteminfo0_item_market_actions"]/a/span[2]').is_displayed():
            g = 0
        price = driver.find_element(By.XPATH,
                                    f'//*[@id="iteminfo{g}_item_market_actions"]/div/div[2]').text
        price = price[3:price.find('у') - 2].replace(',', '.')
        if price[0] == 'ч':
            print('No one is selling this item now!\nSet the price manually: ', end='')
            price = float(input())
        return float(price) + 1

    # функция окна продажи
    def sell_window(self, l, price, current_name=''):
        driver.find_element(By.XPATH, '//*[@id="market_sell_currency_input"]').send_keys(f'{price}')
        if l == 0:
            driver.find_element(By.XPATH, '//*[@id="market_sell_dialog_accept_ssa"]').click()
        driver.find_element(By.XPATH, '//*[@id="market_sell_dialog_accept"]/span').click()
        sleep(1)
        driver.find_element(By.XPATH, '//*[@id="market_sell_dialog_ok"]/span').click()
        sleep(1)
        try:
            if driver.find_element(By.XPATH, '/html/body/div[5]/div[2]/div/div[2]').is_displayed():
                wait_for_conf[current_name] = price
                driver.find_element(By.XPATH, '/html/body/div[5]/div[3]/div/div[2]/div').click()
                return -10
        except:
            pass
        try:
            if driver.find_element(By.XPATH, '//*[@id="market_sell_dialog_error"]').is_displayed():
                print('This item is unavailable or awaiting the confirmation!')
                driver.find_element(By.XPATH, '//*[@id="market_sell_dialog"]/div[2]/div/div').click()
                return -10
        except:
            pass

    # функция кол-ва предметов
    def count_of_items(self):
        count = driver.find_element(By.XPATH,
                                    f'//span[text()="{game_name}"]/following-sibling::span').text.strip(
            '()')
        return int(count)

    # функция продажи по кол-ву
    def sell_by_count(self):
        self.enter_inventory()
        print('How much items you want to sell by quantity? ', end='')
        l = 0
        done = 0
        i = 0
        size = int(input())
        while done != size:
            try:
                list_of_items = driver.find_elements(By.CLASS_NAME, 'inventory_item_link')
                list_of_items[i].click()
                sleep(1)
                i += self.may_or_not()
                price = self.get_price_of_item()
                if price > extreme_price:
                    i += 1
                    continue
                driver.find_element(By.XPATH,
                                    f'//*[@id="iteminfo{g}_item_market_actions"]/a/span[2]').click()
                if self.sell_window(l, price) == -10: i += 1
                l += 1
                done += 1
            except:
                pass
        print('All ready!')

    # функция продажи по типу
    def sell_by_type(self):
        global g
        g = 1
        self.enter_inventory()
        print('What type of items you want to choose? ', end='')
        rec_txt()
        type_of_item = recognize_txt.lower()
        print('How much items you want to sell by type? ', end='')
        size = int(input())
        done = 0
        i = 0
        l = 0
        while size != done:
            try:
                list_of_items = driver.find_elements(By.CLASS_NAME, 'inventory_item_link')
                list_of_items[i].click()
                sleep(1)
                if self.may_or_not():
                    self.is_last(i)
                    i += 1
                    continue
                local_type = driver.find_element(By.XPATH, f'//*[@id="iteminfo{g}_item_type"]').text
                index_of_comma = local_type.find(',')
                if type_of_item.lower() == local_type[:index_of_comma].lower():
                    price = self.get_price_of_item()
                    if price > extreme_price:
                        i += 1
                        continue
                    driver.find_element(By.XPATH,
                                        f'//*[@id="iteminfo{g}_item_market_actions"]/a/span[2]').click()
                    self.sell_window(l, price)
                    done += 1
                    l += 1
                else:
                    if local_type != '':
                        if self.is_last(i) == 33: break
                        i += 1
            except:
                pass
        print('All ready!')

    # функция продажи по имени
    def sell_by_item_name(self):
        global g
        self.enter_inventory()
        print('How much items you want to sell by name? ', end='')
        l = 0
        for _ in range(int(input())):
            try:
                driver.find_element(By.XPATH, '//*[@id="filter_control"]').send_keys(
                    Keys.CONTROL + 'A' + Keys.DELETE)
                print('Enter full item name: ', end='')
                item_name = input()
                if item_name in wait_for_conf:
                    print('This item is already waiting the confirmation!')
                    break
                driver.find_element(By.XPATH, '//*[@id="filter_control"]').send_keys(item_name + Keys.ENTER)
                sleep(1)
                list_of_items = driver.find_elements(By.CLASS_NAME, 'inventory_item_link')
                for i in list_of_items:
                    if i.is_displayed():
                        i.click()
                        break
                sleep(1)
                if self.may_or_not():
                    print("I can't sell this item!")
                    continue
                price = self.get_price_of_item()
                driver.find_element(By.XPATH,
                                    f'//*[@id="iteminfo{g}_item_market_actions"]/a/span[2]').click()
                self.sell_window(l, price, item_name)
                l += 1
            except:
                print('This item is unavailable for sale!')
        sleep(2)
        print('All ready!')
```
### Return Items
```py
#  Класс снятия лота
class ReturnItems:
    def __init__(self):
        global driver

    def lets_return(self):
        driver.get('https://steamcommunity.com/market/')
        items = driver.find_elements(By.CLASS_NAME, 'item_market_action_button_contents')
        l = len(items)
        while l != 1:
            try:
                items[1].click()
                sleep(1)
                driver.find_element(By.ID, 'market_removelisting_dialog_accept').click()
                sleep(1)
                driver.get('https://steamcommunity.com/market/')
                items = driver.find_elements(By.CLASS_NAME, 'item_market_action_button_contents')
                l = len(items)
            except:
                continue
        print('All ready!')
```
### Editting Profile
```py
#  Класс редактирования профлия
class EditingProfile:
    def __init__(self):
        global driver, recognize_txt

    def enter_profile(self):
        driver.find_element(By.XPATH, '//*[@id="global_header"]/div/div[2]/a[3]').click()
        driver.find_element(By.XPATH, '//*[@id="friendactivity_right_column"]/div/div[3]/div[2]/a/span').click()

    def save(self):
        sleep(1)
        try:
            driver.find_element(By.XPATH, '//*[@id="application_root"]/div[2]/div[2]/form/div[7]/button[1]').click()
        except:
            driver.find_element(By.XPATH,
                                '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[3]/button[1]').click()

    def activate_redactor(self):
        self.enter_profile()
        print('Say the parameters you want to change! ', end='')
        rec_txt()
        if 'имя профиля' in recognize_txt.lower() or 'profile name' in recognize_txt.lower(): self.change_profile_name()
        if 'настоящее имя' in recognize_txt.lower() or 'real name' in recognize_txt.lower(): self.change_real_name()
        if 'ссылк' in recognize_txt.lower() or 'personal link' in recognize_txt.lower(): self.change_personal_link()
        if 'о себе' in recognize_txt.lower() or 'about me' in recognize_txt.lower(): self.change_inf_about_me()
        if 'тема' in recognize_txt.lower() or 'topic' in recognize_txt.lower() or 'тему' in recognize_txt.lower(): self.change_topic()

    def change_profile_name(self):
        print('Enter your new profile name: ', end='')
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[3]/div[2]/div[1]/label/div[2]/input').send_keys(
            Keys.CONTROL + 'a', Keys.DELETE)
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[3]/div[2]/div[1]/label/div[2]/input').send_keys(
            input())
        self.save()

    def change_real_name(self):
        print('Enter your new real name: ', end='')
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[3]/div[2]/div[2]/label/div[2]/input').send_keys(
            Keys.CONTROL + 'a', Keys.DELETE)
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[3]/div[2]/div[2]/label/div[2]/input').send_keys(
            input())
        self.save()

    def change_personal_link(self):
        print('Enter your new personal link: ', end='')
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[3]/div[2]/div[3]/label/div[2]/input').send_keys(
            Keys.CONTROL + 'a', Keys.DELETE)
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[3]/div[2]/div[3]/label/div[2]/input').send_keys(
            input())
        self.save()

    def change_inf_about_me(self):
        print('Enter new information about yourself: ', end='')
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[5]/div[2]/textarea').send_keys(
            Keys.CONTROL + 'a', Keys.DELETE)
        driver.find_element(By.XPATH,
                            '//*[@id="application_root"]/div[2]/div[2]/form/div[5]/div[2]/textarea').send_keys(
            input())
        self.save()

    def change_topic(self):
        driver.find_element(By.XPATH, '//*[@id="application_root"]/div[2]/div[1]/a[5]').click()
        sleep(0.5)
        print('Enter new topic name: ', end='')
        dict_of_topics = {'по умолчанию': driver.find_element(By.XPATH,
                                                              '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[2]/div/div/div/div[1]'),
                          'лето': driver.find_element(By.XPATH,
                                                      '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[2]/div/div/div/div[2]'),
                          'полночь': driver.find_element(By.XPATH,
                                                         '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[2]/div/div/div/div[3]'),
                          'сталь': driver.find_element(By.XPATH,
                                                       '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[2]/div/div/div/div[4]'),
                          'космос': driver.find_element(By.XPATH,
                                                        '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[2]/div/div/div/div[5]'),
                          'тьма': driver.find_element(By.XPATH,
                                                      '//*[@id="application_root"]/div[2]/div[2]/div[3]/div/div[2]/div/div/div/div[6]'),
                          }
        command = dict_of_topics[input().lower()]
        command.click()
        self.save()
```
### Delete The Game
```py
# Класс удаления игр
class DelTheGame:
    def __init__(self):
        global driver

    def deleting_game(self):
        driver.get('https://help.steampowered.com/ru/')
        print('Be careful when deleting a game from your account!')
```
### Work With Trade Platform
```py
# Класс работы с торговой площадкой
class WorkWithTradePlatform:
    def __init__(self):
        global driver

    def searching_on_platform(self):
        driver.get('https://steamcommunity.com/market/')
        print('Enter the name of the item you want to view: ', end='')
        driver.find_element(By.XPATH, '//*[@id="findItemsSearchBox"]').send_keys(input())
        driver.find_element(By.XPATH, '//*[@id="findItemsSearchSubmit"]').click()
```
### Launch Bot
```py
#  Класс запуска
class LaunchBot(SaleOfItems, ReturnItems, EditingProfile, TradeOffer, WorkWithTradePlatform, DelTheGame):
    pass


confirm_to_steam()

conv = ['Всё хорошо. А вы как?', 'Ждал вас и дождался. Вот бы всегда так было.', 'Отлично. Правда, немного одиноко.',
        'У меня все хорошо. Надеюсь у вас тоже.', 'Да норм, если честно.',
        'Отлично, приятно, что интересуетесь. У вас как?',
        'Сегодня ничего не произошло. Сидел у воображаемого окна. Думал о вас.']
thanks = ['И вам спасибо, вы очень хороший человек.', 'Люблю свою работу, спасибо.',
          'Это было не трудно, это по любви.', 'У меня даже заискрило от радости.', 'Вам спасибо. Мы все молодцы.']
hello_words = ['Привет! А мы не виделись сто лет.',
               'Привет! Как же я вас ждал!', 'Я здесь.', 'Хеллоу.']
               
```
### Greeting
```py
#  Приветствие
def start_text():
    list_of_sale_com = ['Sell / By quantity', 'Sell / By type / <Type of item>',
                        'Sell / By item name']
    edit_com = 'Profile / Profile name + Real name + Personal link + About me + Topic'
    print(f'Hi, {login}! My name is Mark. I will be your assistant in working with the Steam system!')
    sleep(1)
    print('To get a list of commands, input "get"!')
    sleep(1)
    print('To start working with me, press ENTER!')
    if input() == 'get':
        print('Close the application ---> Exit')
        for i in range(len(list_of_sale_com)):
            slash_index = len(list_of_sale_com[i])
            if '/' in list_of_sale_com[i][list_of_sale_com[i].rfind('B'):slash_index]:
                slash_index = list_of_sale_com[i].rfind('/')
            print(f"Sale {list_of_sale_com[i][list_of_sale_com[i].rfind('B'):slash_index]}", ' ---> ',
                  list_of_sale_com[i])
        print('Editing profile ---> ', edit_com, '\nCreate new trade offer ---> Trade offer',
              '\nRemoving lots ---> Remove lots', '\nOpen trading platform ---> Trade platform',
              '\nDeleting the game ---> Delete game')
        w = input()


start_text()
```
### Command Reading Cycle
```py
bot = LaunchBot()
while ('exit' not in recognize_txt) and ('выйти' not in recognize_txt):
    try:
        print(f'Say the command:', end=' ')
        rec_txt()
        if 'sell' in recognize_txt.lower() or 'прода' in recognize_txt.lower():
            print(f'How you want to sell your items?', end=' ')
            rec_txt()
            if 'quantity' in recognize_txt.lower() or 'количеств' in recognize_txt.lower():
                bot.sell_by_count()
            if 'type' in recognize_txt.lower() or 'тип' in recognize_txt.lower():
                bot.sell_by_type()
            if 'name' in recognize_txt.lower() or 'имен' in recognize_txt.lower():
                bot.sell_by_item_name()
        if 'profile' in recognize_txt.lower() or 'профил' in recognize_txt.lower():
            bot.activate_redactor()
        if 'trade' in recognize_txt or 'обмен' in recognize_txt:
            bot.create_new_trade_offer()
        if 'лот' in recognize_txt.lower() or 'remove' in recognize_txt.lower() or 'верни' in recognize_txt.lower():
            bot.lets_return()
        if 'как' in recognize_txt.lower() and 'дела' in recognize_txt.lower():
            print(choice(conv))
        if 'привет' in recognize_txt.lower():
            print(choice(hello_words))
        if 'молодец' in recognize_txt.lower():
            print(choice(thanks))
        if 'platform' in recognize_txt.lower() or 'торгов' in recognize_txt.lower() and 'площадк' in recognize_txt.lower():
            bot.searching_on_platform()
        if 'delet' in recognize_txt.lower() or 'удал' in recognize_txt.lower() and 'игр' in recognize_txt.lower():
            bot.deleting_game()
    except:
        pass

```
---
[Go To TOP](#TOP)
