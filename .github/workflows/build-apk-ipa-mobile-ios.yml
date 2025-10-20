import flet
from flet import Page, Text, ElevatedButton, Column, Row, Dropdown, dropdown, AlertDialog, TextField, Container
import datetime
import json
import os

DATA_FILE = "attendance_flet.json"

if not os.path.exists(DATA_FILE):
    with open(DATA_FILE, 'w') as f:
        json.dump({}, f)

def load_data():
    with open(DATA_FILE, 'r') as f:
        return json.load(f)

def save_data(data):
    with open(DATA_FILE, 'w') as f:
        json.dump(data, f, indent=2)

def get_daily_salary(date_obj):
    return 225 if date_obj.weekday() == 4 else 175  # الجمعة = 4

def main(page: Page):
    page.title = "روزن - سجل الحضور"
    page.vertical_alignment = "start"
    page.padding = 20

    today = datetime.date.today()
    current_year = today.year
    current_month = today.month

    data = load_data()

    title = Text("روزن", size=40, weight="bold", color="#D2691E")
    page.add(title)

    years_range = list(range(today.year - 5, today.year + 6))
    year_dropdown = Dropdown(
        label="السنة",
        width=150,
        options=[dropdown.Option(str(y)) for y in years_range],
        value=str(current_year)
    )

    months_names = ["يناير", "فبراير", "مارس", "أبريل", "مايو", "يونيو",
                    "يوليو", "أغسطس", "سبتمبر", "أكتوبر", "نوفمبر", "ديسمبر"]
    month_dropdown = Dropdown(
        label="الشهر",
        width=150,
        options=[dropdown.Option(f"{i+1} - {months_names[i]}") for i in range(12)],
        value=f"{current_month} - {months_names[current_month - 1]}"
    )

    days_column = Column()
    total_text = Text("", size=18, weight="bold", color="green")

    def get_days_in_month(year, month):
        if month == 12:
            next_month = datetime.date(year + 1, 1, 1)
        else:
            next_month = datetime.date(year, month + 1, 1)
        return (next_month - datetime.timedelta(days=1)).day

    def refresh_days():
        days_column.controls.clear()
        y = int(year_dropdown.value)
        m = int(month_dropdown.value.split(" - ")[0])
        days_in_month = get_days_in_month(y, m)

        def day_button_click(day):
            date_str = f"{y}-{m:02d}-{day:02d}"
            date_obj = datetime.date(y, m, day)

            def open_edit_dialog():
                salary_field.value = str(data[date_str])
                dlg_modal.open = True
                page.update()

                def on_save_click(e):
                    try:
                        new_salary = float(salary_field.value)
                        if new_salary < 0:
                            page.snack_bar = flet.SnackBar(Text("السعر لازم يكون موجب"))
                            page.snack_bar.open = True
                            page.update()
                            return
                        data[date_str] = new_salary
                        save_data(data)
                        refresh_days()
                        update_total()
                        dlg_modal.open = False
                        page.update()
                    except:
                        page.snack_bar = flet.SnackBar(Text("ادخل رقم صحيح"))
                        page.snack_bar.open = True
                        page.update()

                def on_delete_click(e):
                    if date_str in data:
                        del data[date_str]
                        save_data(data)
                        refresh_days()
                        update_total()
                    dlg_modal.open = False
                    page.update()

                save_button.on_click = on_save_click
                delete_button.on_click = on_delete_click

            if date_str not in data:
                salary = get_daily_salary(date_obj)
                data[date_str] = salary
                save_data(data)
                refresh_days()
                update_total()
                open_edit_dialog()
            else:
                open_edit_dialog()

        row_controls = []
        for day in range(1, days_in_month + 1):
            date_str = f"{y}-{m:02d}-{day:02d}"
            salary = data.get(date_str)
            date_obj = datetime.date(y, m, day)
            btn_text = f"{day}"

            if salary is not None:
                btn_text += f"\n{int(salary)} ج"
                btn_color = "#90EE90"
            elif date_obj.weekday() == 4:
                btn_color = "#FFFACD"
            else:
                btn_color = "#FFFFFF"

            btn = ElevatedButton(
                text=btn_text,
                width=50,
                height=50,
                bgcolor=btn_color,
                on_click=lambda e, d=day: day_button_click(d)
            )
            row_controls.append(btn)
            if len(row_controls) == 7:
                days_column.controls.append(Row(row_controls, spacing=5))
                row_controls = []
        if row_controls:
            days_column.controls.append(Row(row_controls, spacing=5))

        page.update()

    def update_total():
        y = int(year_dropdown.value)
        m = int(month_dropdown.value.split(" - ")[0])
        total_days = 0
        total_salary = 0
        for date_str, salary in data.items():
            dt = datetime.datetime.strptime(date_str, "%Y-%m-%d").date()
            if dt.year == y and dt.month == m:
                total_days += 1
                total_salary += salary
        total_text.value = f"📊 عدد الأيام: {total_days} - الإجمالي: {total_salary} جنيه"
        page.update()

    # ✅ تعديل يوم معين تطلبه انت
    def edit_custom_day(e):
        def on_submit_day(e):
            try:
                day_num = int(day_input.value)
                y = int(year_dropdown.value)
                m = int(month_dropdown.value.split(" - ")[0])
                days_in_month = get_days_in_month(y, m)

                if not (1 <= day_num <= days_in_month):
                    raise ValueError("خارج نطاق الشهر")

                date_str = f"{y}-{m:02d}-{day_num:02d}"
                if date_str not in data:
                    page.snack_bar = flet.SnackBar(Text("❗ اليوم غير مسجل بعد!"))
                    page.snack_bar.open = True
                    page.update()
                    return

                # افتح نافذة التعديل
                salary_field.value = str(data[date_str])
                dlg_modal.open = True
                day_dialog.open = False
                page.update()

                def on_save_click(e):
                    try:
                        new_salary = float(salary_field.value)
                        data[date_str] = new_salary
                        save_data(data)
                        refresh_days()
                        update_total()
                        dlg_modal.open = False
                        page.update()
                    except:
                        page.snack_bar = flet.SnackBar(Text("❌ ادخل رقم صحيح"))
                        page.snack_bar.open = True
                        page.update()

                def on_delete_click(e):
                    del data[date_str]
                    save_data(data)
                    refresh_days()
                    update_total()
                    dlg_modal.open = False
                    page.update()

                save_button.on_click = on_save_click
                delete_button.on_click = on_delete_click

            except:
                page.snack_bar = flet.SnackBar(Text("❗ ادخل رقم يوم صحيح"))
                page.snack_bar.open = True
                page.update()

        day_input = TextField(label="رقم اليوم", keyboard_type="number", width=150)
        day_dialog = AlertDialog(
            modal=True,
            title=Text("اختيار يوم للتعديل"),
            content=day_input,
            actions=[
                ElevatedButton("التالي", on_click=on_submit_day),
                ElevatedButton("إلغاء", on_click=lambda e: day_dialog.close())
            ]
        )
        page.dialog = day_dialog
        day_dialog.open = True
        page.update()

    year_dropdown.on_change = lambda e: (refresh_days(), update_total())
    month_dropdown.on_change = lambda e: (refresh_days(), update_total())

    salary_field = TextField(label="السعر", width=200, keyboard_type="number")
    save_button = ElevatedButton("حفظ", on_click=lambda e: None)
    delete_button = ElevatedButton("حذف", on_click=lambda e: None)

    dlg_modal = AlertDialog(
        modal=True,
        title=Text("تعديل السعر"),
        content=salary_field,
        actions=[
            save_button,
            delete_button,
            ElevatedButton("إلغاء", on_click=lambda e: dlg_modal.close())
        ],
    )

    page.dialog = dlg_modal

    controls = Column([
        Row([year_dropdown, month_dropdown], alignment="start", spacing=20),
        Container(content=Text("اختر يوم العمل:", size=16, weight="bold"), margin=10),
        days_column,
        total_text,
        ElevatedButton("✏️ تعديل سعر يوم معين", on_click=edit_custom_day, bgcolor="#FFA07A", color="white")
    ])

    page.add(controls)
    refresh_days()
    update_total()

flet.app(target=main)
