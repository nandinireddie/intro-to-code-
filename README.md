from tkinter import *
from tkinter import messagebox, simpledialog
import sqlite3 as sql
import datetime

def add_task():
    task_string = task_field.get()
    priority_level = priority_combo.get()
    status = status_combo.get()

    if len(task_string) == 0:
        messagebox.showinfo('Error', 'Field is Empty.')
    else:
        current_time = datetime.datetime.now()
        formatted_time = current_time.strftime("%Y-%m-%d %H:%M:%S")
        task_with_time = f"{task_string} (added at {formatted_time})"

        tasks.append((task_with_time, priority_level, status))
        the_cursor.execute('insert into tasks values (?, ?, ?)', (task_with_time, priority_level, status))
        list_update()

        task_field.delete(0, 'end')
        priority_combo.delete(0, 'end')
        status_combo.delete(0, 'end')

def list_update():
    task_listbox.delete(0, 'end')
    for task in tasks:
        task_listbox.insert('end', task)


def edit_task():
    try:
        selected_task = task_listbox.get(task_listbox.curselection())
        if selected_task in tasks:
            new_title = simpledialog.askstring("Edit Task", f"Edit task '{selected_task}' title to:")
            new_priority = simpledialog.askstring("Edit Task Priority",
                                                  f"Enter new priority for task '{selected_task}':")
            new_status = simpledialog.askstring("Edit Task Status", f"Enter new status for task '{selected_task}':")

            if new_title is not None and new_priority is not None and new_status is not None:
                current_time = datetime.datetime.now()
                formatted_time = current_time.strftime("%Y-%m-%d %H:%M:%S")
                new_title1 = f"{new_title} (edited at {formatted_time})"

                index = tasks.index(selected_task)
                tasks[index] = (new_title1, new_priority, new_status)

                list_update()
                the_cursor.execute('UPDATE tasks SET title = ?, priority = ?, status = ? WHERE title = ?',
                                   (new_title1, new_priority, new_status, selected_task))
    except:
        messagebox.showinfo('Error', 'No Task Selected. Cannot Edit.')

    list_update()

    clear_list()
    for task in tasks:
        task_listbox.insert('end', task)


def delete_task():
    try:
        the_value = task_listbox.get(task_listbox.curselection())
        if the_value in tasks:
            tasks.remove(the_value)

            list_update()
            the_cursor.execute('delete from tasks where title = ?', (the_value,))
    except:
        messagebox.showinfo('Error', 'No Task Selected. Cannot Delete.')


def delete_all_tasks():
    message_box = messagebox.askyesno('Delete All', 'Are you sure?')
    if message_box == True:
        while len(tasks) > 0:
            tasks.pop()
        the_cursor.execute('delete from tasks')

        list_update()


def clear_list():
    task_listbox.delete(0, 'end')


def close():
    print(tasks)
    guiWindow.destroy()


def retrieve_database():
    while (len(tasks) != 0):
        tasks.pop()
    for row in the_cursor.execute('select title from tasks'):
        tasks.append(row)

if __name__ == "__main__":
    guiWindow = Tk()
    guiWindow.title("Task Manager App")
    guiWindow.geometry("665x400+550+250")
    guiWindow.resizable(0, 0)
    guiWindow.configure(bg="WHITE")

    the_connection = sql.connect('task.db')
    the_cursor = the_connection.cursor()
    the_cursor.execute("DROP TABLE IF EXISTS tasks")
    the_cursor.execute('CREATE TABLE IF NOT EXISTS tasks (title TEXT, status TEXT, priority TEXT)')
    tasks = []

    functions_frame = Frame(guiWindow, bg="WHITE")
    functions_frame.pack(side="top", expand=True, fill="both")

    task_label = Label(functions_frame, text="Enter Task:", font=("Arial", 14, "bold"), background="WHITE")
    task_label.place(x=20, y=30)
    task_field = Entry(functions_frame, font=("Arial", 14), width=43, foreground="black", background="white")
    task_field.place(x=180, y=30)

    priority_label = Label(functions_frame, text="Priority Level:", font=("Arial", 14, "bold"), background="WHITE")
    priority_label.place(x=20, y=60)
    priority_combo = Entry(functions_frame, font=("Arial", 14), width=20, foreground="black", background="white")
    priority_combo.place(x=180, y=60)

    status_label = Label(functions_frame, text="Status:", font=("Arial", 14, "bold"), background="WHITE")
    status_label.place(x=360, y=60)
    status_combo = Entry(functions_frame, font=("Arial", 14), width=17, foreground="black", background="white")
    status_combo.place(x=460, y=60)

    add_button = Button(functions_frame, text="Add Task", width=15, bg='green', fg='white', font=("Arial", 14, "bold"), command=add_task)
    add_button.place(x=2, y=90)

    edit_button = Button(functions_frame, text="Edit Task", width=15, bg='green', fg='white', font=("Arial", 14, "bold"), command=edit_task)
    edit_button.place(x=180, y=90)

    del_button = Button(functions_frame, text="Delete Task", width=15, bg='green', fg='white', font=("Arial", 14, "bold"), command=delete_task)
    del_button.place(x=360, y=90)

    del_all_button = Button(functions_frame, text="Delete All", width=14, bg='green', fg='white', font=("Arial", 14, "bold"), command=delete_all_tasks)
    del_all_button.place(x=520, y=90)

    exit_button = Button(functions_frame, text="Exit", width=52, bg='green', fg='white', font=("Arial", 14, "bold"), command=close)
    exit_button.place(x=17, y=330)

    task_listbox = Listbox(functions_frame, width=57, height=7, font="bold", selectmode='SINGLE', background="WHITE", foreground="BLACK", selectbackground="#D4AC0D", selectforeground="BLACK")
    task_listbox.place(x=17, y=140)

    retrieve_database()
    list_update()

    guiWindow.mainloop()

    the_connection.commit()
    the_cursor.close()
