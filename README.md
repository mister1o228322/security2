Домашнее задание к занятию «Защита хоста»
# Задание 1
1. Установка eCryptfs

Был установлен пакет ecryptfs-utils и необходимые зависимости:

sudo apt update

sudo apt install ecryptfs-utils -y

2. Создание пользователя cryptouser

Создан новый пользователь с домашним каталогом:


sudo adduser cryptouser

Был задан пароль для пользователя (например, cryptopass).

3. Подготовка тестовых файлов
   
Под пользователем cryptouser созданы файлы с конфиденциальной информацией для последующей проверки шифрования:


su - cryptouser

echo "Это секретный пароль от банка: 123456789" > bank_password.txt

echo "Логин: admin, пароль: secret123" > credentials.txt

mkdir private

echo "Конфиденциальный документ" > private/doc.txt

exit

4. Шифрование домашнего каталога

Выполнена миграция домашнего каталога пользователя cryptouser с использованием eCryptfs:

sudo ecryptfs-migrate-home -u cryptouser

В результате был создан зашифрованный каталог, а исходные данные перемещены в резервную копию /home/cryptouser.uqIwnmVb.

5. Создание ключа восстановления

Сгенерирован ключ для возможности восстановления данных в случае необходимости:

sudo ecryptfs-unwrap-passphrase

Ключ сохранен в безопасном месте.

Результаты

<img width="901" height="830" alt="image" src="https://github.com/user-attachments/assets/9c50d787-78f1-4cb6-b270-a72f4d06722c" />
<img width="679" height="250" alt="image" src="https://github.com/user-attachments/assets/0f108e13-6788-4d61-97da-4726a3b52548" />
<img width="753" height="381" alt="image" src="https://github.com/user-attachments/assets/95a9357b-5f24-47f8-b5fa-101219505d29" />
<img width="596" height="189" alt="image" src="https://github.com/user-attachments/assets/adba89fe-9029-4124-b943-8bf721459738" />
<img width="1296" height="881" alt="image" src="https://github.com/user-attachments/assets/419fbbeb-056d-46a3-80cc-3bd63379435f" />
<img width="874" height="893" alt="image" src="https://github.com/user-attachments/assets/fc0a1fdc-0f9e-4319-9f8f-0b732306c7fa" />

# Задание 2

1. Установка cryptsetup
Был установлен пакет cryptsetup, необходимый для работы с LUKS:


sudo apt update

sudo apt install cryptsetup -y

2. Создание файла-образа для раздела

Для имитации отдельного раздела был создан файл размером 100 МБ, заполненный нулями:

dd if=/dev/zero of=luks_test.img bs=1M count=100

В результате был получен файл luks_test.img размером 100 МБ.

3. Привязка к loop-устройству

Созданный файл был привязан к виртуальному блочному устройству, чтобы система воспринимала его как настоящий диск:

sudo losetup -fP luks_test.img

losetup -l | grep luks_test

В результате файл стал доступен как устройство /dev/loop0.

4. Инициализация LUKS (шифрование раздела)

На устройстве /dev/loop0 была выполнена инициализация LUKS, которая создает заголовок шифрования и подготавливает раздел к использованию:

sudo cryptsetup luksFormat /dev/loop0

Было подтверждено действие (введено YES заглавными буквами) и задан пароль для доступа к зашифрованному разделу.

5. Открытие зашифрованного раздела
   
Зашифрованный раздел был открыт и сопоставлен с устройством /dev/mapper/my_encrypted_volume:

sudo cryptsetup luksOpen /dev/loop0 my_encrypted_volume

При этом был запрошен пароль, заданный на предыдущем шаге. После успешного ввода пароля раздел стал доступен как блочное устройство.

6. Создание файловой системы

На открытом (расшифрованном) устройстве была создана файловая система ext4:

sudo mkfs.ext4 /dev/mapper/my_encrypted_volume

7. Монтирование и создание тестовых файлов

Раздел был смонтирован в каталог /mnt/encrypted, после чего на нем были созданы тестовые файлы с конфиденциальной информацией:

sudo mkdir -p /mnt/encrypted

sudo mount /dev/mapper/my_encrypted_volume /mnt/encrypted

echo "Секретные данные LUKS - пароль базы данных: dbpass123" | sudo tee /mnt/encrypted/secret.txt

echo "Логины от серверов: root:rootpass, admin:admin123" | sudo tee /mnt/encrypted/passwords.txt

Было проверено содержимое раздела:

sudo ls -la /mnt/encrypted/

В выводе присутствовали файлы secret.txt и passwords.txt, а также системный каталог lost+found. Содержимое файлов соответствовало введенным данным.

8. Демонстрация зашифрованного состояния

Раздел был размонтирован и закрыт:

sudo umount /mnt/encrypted

sudo cryptsetup luksClose my_encrypted_volume

После закрытия было выполнено прямое чтение зашифрованного устройства:

sudo hexdump -C /dev/loop0 | head -30

Вывод команды hexdump показал нечитаемые данные (случайный набор байт в шестнадцатеричном представлении), что подтверждает факт шифрования. Никакой структуры файловой системы или читаемого текста обнаружено не было.

9. Проверка восстановления доступа

Для подтверждения целостности данных раздел был снова открыт и смонтирован:

sudo cryptsetup luksOpen /dev/loop0 my_encrypted_volume

sudo mount /dev/mapper/my_encrypted_volume /mnt/encrypted

ls -la /mnt/encrypted/

cat /mnt/encrypted/secret.txt

Все файлы оказались на месте, их содержимое полностью соответствовало исходному. Это подтверждает, что шифрование и дешифрование работают корректно и данные не были повреждены.

Скриншоты:

<img width="1342" height="265" alt="image" src="https://github.com/user-attachments/assets/132c1b4f-0d9e-40b2-ba2f-3dad04bc7d73" />
<img width="566" height="111" alt="image" src="https://github.com/user-attachments/assets/6432a7f0-16ad-4521-956e-927b3e65ef3c" />
<img width="688" height="193" alt="image" src="https://github.com/user-attachments/assets/0bca7882-457f-4b0f-be69-aa99d8153be0" />
<img width="658" height="406" alt="image" src="https://github.com/user-attachments/assets/03b101f8-748e-4394-982d-6b58b60e0d86" />
<img width="791" height="1216" alt="image" src="https://github.com/user-attachments/assets/4347830e-046d-4fb4-8810-442a027b26bf" />
<img width="791" height="1216" alt="image" src="https://github.com/user-attachments/assets/4b050695-fc80-4287-96e5-780b9e9458de" />



