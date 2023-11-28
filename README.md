## Тема: Конвертер изображений 
### Это процесс конвертирования какого-либо загруженного изображения из одного формата в другой. Ниже представлен мой код программы и также прикреплен отдельный exe-файл. У программы есть несколько виджетов:
#### - Загрузка изображения - местом для вставки я использовал Label.
#### - Обрезка (Редактировать) - обрезка идет по координатам, начиная с центра изображения.
#### - Наложение фильтра - накладывает фильтр на изображение и выводит диалоговое окно с вопросом о сохранении или нет.
#### - Конвертировать - Конвертирование изображение (необязательно вписывать расширения, однако выводит с ошибкой JPG/JPEG файлы, будет дорабатываться).
#### Когда вы откроете ветку с моим exe, оно напишет что файл слишком большой, просто нажмите на View raw и оно все равно должно скачаться.
https://prezi.com/view/kltdKweGryenXfiXHxJQ/ - презентация моей темы, использовался отдельный источник для её создания (prezi.com).

```py
from tkinter import * #Создание labels, кнопок и так далее
from tkinter import filedialog #Открывает проводник через который мы загружаем изображение и применяем через него формат/расширение файла.
from tkinter import messagebox #Диалоговые окна для предоставления информации о том что было сделано, добавлено, отменено.
from PIL import Image, ImageTk, ImageFilter #Библиотека PIL в основном нужна для загрузки изображения на главное окно а также для наложения фильтров.
import tkinter.simpledialog #Позволяет создать диалоговое окно в котором программа спрашивает пользователя о вводе данных (в данном случае координаты нашего изображения)
from tkinter.ttk import Combobox #Открывает выбор того что мы хотим взять. Применяется для фильтров

#Функция zagruzit() используется для загрузки изображения на главное окно в Label6.
def zagruzit():
    global loaded, current, label6 #Используем global для обозначения переменных в одном коде в каждой функции.

    #Создаем переменную path в которой размещаем пути которые доступны для загрузки изображения.
    path = filedialog.askopenfilename(filetypes=[("Image files", "*.jpg *.png *.jpeg *.gif *.bmp *.tiff *.mp4")])
    if path: #Если такое расширение изображение существует то оно загружается на главное окно.
        loaded = Image.open(path)
        current = ImageTk.PhotoImage(loaded)
        label6.config(image=current)

#Функция konv() позволяет пользователю конвертировать изображение в любой доступный формат.
def konv():
    if loaded:
        choice = [("JPEG", "*.jpg"), ("PNG", "*.png"), ("JPEG 2000", "*.jpeg"), ("WEBP", "*.webp"),
                  ("GIF", "*.gif"), ("PDF", "*.pdf")]
        chosen_file = filedialog.asksaveasfile(filetypes=choice, defaultextension=".jpg")
        if chosen_file is not None:
            chosen_filename = chosen_file.name
            format_name = chosen_file.name.split(".")[-1].lower()  # Она разделяет имя файла по точке . и выбирает последний элемент списка с помощью [-1]. Затем метод lower() применяется для преобразования расширения файла в нижний регистр.
            if format_name in [fmt[1][2:] for fmt in choice]:  # В этой части кода мы обращаемся к элементу с индексом 1 в кортеже fmt. Затем на этом элементе применяем оператор среза [2:], чтобы получить подстроку, начиная с третьего символа до конца строки.
                loaded.save(chosen_filename, format=format_name)
                messagebox.showinfo("Успех", "Изображение успешно конвертировано и сохранено.")
            else:
                messagebox.showerror("Ошибка", "Выбран неподдерживаемый формат файла.")

#Функция nal_filtr() отвечает за наложение фильтра на изображение
def nal_filtr():
    if loaded:
        win_filtr = Toplevel(root) #Создание дополнительного окна с Фильтрами.
        win_filtr.title("Выберите фильтр")
        win_filtr.geometry("250x100")
        win_filtr.transient(root)

        #Дается выбор фильтра который будет отображаться с помощью Combobox
        options = ["DETAIL", "BLUR", "CONTOUR", "SMOOTH"]
        chosen_filtr = Combobox(win_filtr, values=options)
        chosen_filtr.pack(pady=10)

        #Создается кнопка "Применить фильтр" которая привязана с функцией primenit_filtr().
        apply = Button(win_filtr, text="Применить фильтр", bg="gray", fg="white", font=('Times New Roman', 15, 'bold'),
                       command=lambda: primenit_filtr(chosen_filtr.get()))
        apply.pack(pady=10)

#Функция primenit_filtr() нужна чтобы фильтры успешно накладывались на изображение. Для этого у нас есть библиотека PIL
#В которую мы добавляем модуль ImageFilter и используем его для каждого фильтра.
def primenit_filtr(type):
    global loaded, current, label6
    if loaded:
        filtr_image = None
        if type == "DETAIL":
            filtr_image = loaded.filter(ImageFilter.DETAIL)
        elif type == "BLUR":
            filtr_image = loaded.filter(ImageFilter.BLUR)
        elif type == "CONTOUR":
            filtr_image = loaded.filter(ImageFilter.CONTOUR)
        elif type == "SMOOTH":
            filtr_image = loaded.filter(ImageFilter.SMOOTH)

        #Применение фильтра и отображение измененного изображения на главном окне.
        if filtr_image:
            current = ImageTk.PhotoImage(filtr_image)
            label6.config(image=current)

            #Перед сохранением выводится диалоговое окно для окончательного ответа от пользователя.
            res = messagebox.askyesno("Сохранение", "Желаете сохранить измененное изображение?")
            if res:
                # После нажатия на кнопку Применить фильтр, выскакивает проводник который по факту является тем же Конвертером и конвертирует измененный результат в доступное расширение.
                save_path = filedialog.asksaveasfilename(initialdir="/", title="Сохранить измененное изображение",
                                                         filetypes=(("PNG", ".png"), ("JPEG", ".jpg;.jpeg"),
                                                                    ("WEBP", ".webp"), ("GIF", ".gif"),
                                                                    ("All files", ".")))
                #Если формат выбран, изображение сохраняется
                if save_path:
                    filtr_image.save(save_path)
                    messagebox.showinfo("Успех", "Изображение успешно конвертировано и сохранено.")


#Создание функции red(), которая отвечает за обрезание нашего изображения по координатам. Координаты идут от центра изображения. При нажатии на кнопку, будет поочередно выводиться окна ввода.
def red():
    global loaded, current, label6
    if loaded: #С помощью параметра parent, мы связываем всплывающие окна с главным окном чтобы они всплывались над ним, а не за ним.
        x = tkinter.simpledialog.askinteger("Обрезать изображение", "Введите координату верхнего левого угла X:",
                                            parent=root)
        y = tkinter.simpledialog.askinteger("Обрезать изображение", "Введите координату верхнего левого угла Y:",
                                            parent=root)
        bottomx = tkinter.simpledialog.askinteger("Обрезать изображение", "Введите координату нижнего правого угла X:",
                                                  parent=root)
        bottomy = tkinter.simpledialog.askinteger("Обрезать изображение", "Введите координату нижнего правого угла Y:",
                                                  parent=root)

        # Если пользователь хотя бы раз нажал на кнопку "Отменить" в любом порядке, то программа в конце отменит обрезание.
        if x is not None and y is not None and bottomx is not None and bottomy is not None:
            try:
                #С помощью метода crop() изображение обрезается по введенным нами координатам.
                crop_img = loaded.crop((x, y, bottomx, bottomy))
                current = ImageTk.PhotoImage(crop_img)
                label6.config(image=current)
                loaded = crop_img
            except ValueError: #Выводит ошибку в случае если координаты были неправильно введены.
                messagebox.showerror("Ошибка", "Неверно заданы координаты обрезки.")
        else:
            # И выведет это если пользователь отменяет действие.
            messagebox.showinfo("Информация", "Отменено пользователем.")


def res_change():
    global loaded, current
    new_size = tkinter.simpledialog.askstring("Введите новый размер", "Введите новый размер изображения в формате (ширина, высота):")
    if new_size:
        new_size = tuple(map(int, new_size.split(',')))
        loaded = loaded.resize(new_size)
        current = ImageTk.PhotoImage(loaded)
        label6.config(image=current)

def exit_conv():
    root.destroy()

#Создаем главное окно.
root = Tk()
root.geometry("1281x1025")
root.title("Конвертер Изображений от колледжа им. П.А. Овчинникова")
#root.iconbitmap('pk3.ico')
root.image = ImageTk.PhotoImage(file='doski2.png')
root.resizable(width=False, height=False)
logo = Label(root, image=root.image)
logo.grid(row=0, column=0)

label6 = Label(root, bg="black") #Label который применяется в качестве места для вставки изображение.
label6.grid(row=0, column=0)

label5 = Label(root, text="Онлайн Конвертер \nим. П.К. Овчинникова", fg="white", font="Times 20 bold",
                        relief=RAISED, bg="gray")
label5.grid(row=0, column=0, sticky="nw", pady=50, padx=60)

label1 = Label(root, width=45, height=4, bg="black")
label1.grid(row=0, column=0, sticky="ne", pady=54, padx=37)

label2 = Label(root, width=33, height=4, bg="black")
label2.grid(row=0, column=0, sticky="se", pady=54, padx=37)

label3 = Label(root, width=32, height=4, bg="black")
label3.grid(row=0, column=0, sticky="ne", pady=54, padx=365)

label4 = Label(root, width=18, height=4, bg="black")
label4.grid(row=0, column=0, stick="ne", pady=130, padx=40)

label7 = Label(root, width=35, height=4, bg="black")
label7.grid(row=0, column=0, stick="se", pady=54, padx=285)

label8 = Label(root, width=16, height=4, bg="black")
label8.grid(row=0, column=0, stick="sw", pady=54, padx=48)

#Создаем множество кнопок и присваиваем им команды, то есть наши функции в который содержится остальной код.
load = Button(root, text="Загрузить изображение", bg="gray", fg="white", font=('Times New Roman', 20, 'bold'),command=zagruzit)
load.grid(row=0, column=0, sticky="ne", pady=60, padx=40)

konvert = Button(root, text="Конвертировать", bg="gray", fg="white", font='Times 20 bold', command=konv)
konvert.grid(row=0, column=0, sticky="se", pady=60, padx=40)

filtr = Button(root, text='Фильтр', bg="gray", fg="white", font='Times 20 bold', command=nal_filtr)
filtr.grid(row=0, column=0, stick="ne", pady=135, padx=44)

edit = Button(root, text="Редактировать", bg="gray", fg="white", width=13, font='Times 20 bold',command=red)
edit.grid(row=0, column=0, stick="ne", pady=60, padx=370)

resolution = Button(root, text="Изменить размер", bg="gray", fg="white", font='Times 20 bold', command=res_change)
resolution.grid(row=0, column=0, stick="se", pady=60, padx=291)

exit = Button(root, text="Выход", bg="gray", fg="white", font='Times 20 bold', command=exit_conv)
exit.grid(row=0, column=0, stick="sw", pady=60, padx=55)

root.mainloop()```
