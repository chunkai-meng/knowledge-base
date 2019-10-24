# Django后台导出Excel数据并下载

```python

from django.contrib import admin
from .models import Debit
from django.http import StreamingHttpResponse
import xlwt
import os
@admin.register(Debit)
class DebitAdmin(admin.ModelAdmin):
    # other statements
    actions = ["SaveExecl",]
    def SaveExecl(self, request, queryset):
        Begin = xlwt.Workbook()
        # some sentences
        Begin.save("%s"%(filename))
        def file_iterator(filename, chunk_size=512):
            with open(filename,'rb') as f:
                while True:
                    c = f.read(chunk_size)
                    if c:
                        yield c
                    else:
                        break
        response = StreamingHttpResponse(file_iterator(filename))
        response['Content-Type'] = 'application/octet-stream'
        response['Content-Disposition'] = 'attachment; filename="{}" '.format("Result.xls")
        return response
    SaveExecl.short_description = "以表格形式下载"
```


### 测试成功案例

**views.py**
```python
import xlwt
from django.http import HttpResponse
from django.contrib.auth.models import User
def export_users_xls(request):
    response = HttpResponse(content_type='application/ms-excel')
    response['Content-Disposition'] = 'attachment; filename="users.xls"'
    wb = xlwt.Workbook(encoding='utf-8')
    ws = wb.add_sheet('Users')
    # Sheet header, first row
    row_num = 0
    font_style = xlwt.XFStyle()
    font_style.font.bold = True
    columns = ['Username', 'First name', 'Last name', 'Email address', ]
    for col_num in range(len(columns)):
        ws.write(row_num, col_num, columns[col_num], font_style)
    # Sheet body, remaining rows
    font_style = xlwt.XFStyle()
    rows = User.objects.all().values_list('username', 'first_name', 'last_name', 'email')
    for row in rows:
        row_num += 1
        for col_num in range(len(row)):
            ws.write(row_num, col_num, row[col_num], font_style)
    wb.save(response)
    return response
```

**urls.py**
```
import views
urlpatterns = [
    ...
    url(r'^export/xls/$', views.export_users_xls, name='export_users_xls'),
]
```
[参考](https://www.django.cn/article/show-9.html)

### 可行方案
#### 1. xlwt

```python
import xlwt

def export_transaction_xls(request):
    project_id = request.GET.get('selected_id')
    date_from = request.GET.get('date_from')
    date_to = request.GET.get('date_to')
    date_range = [date_from, date_to]
    queryset = Transaction.objects.day_to_day_archive(date_range).filter(project=project_id)
    metrics = {
        'total_car': Sum('num_car'),
        'total_cube': Sum('num_cube'),
        'total_price': Sum('total_price')
    }

    file_name = f'{date_from}到{date_to}'
    response = HttpResponse(content_type='application/ms-excel')
    response['Content-Disposition'] = f'attachment; filename="{file_name}.xls"'
    wb = xlwt.Workbook(encoding='utf-8')
    ws = wb.add_sheet('生产日报表')
    # Sheet header, first row
    row_num = 0
    font_style = xlwt.XFStyle()
    font_style.font.bold = True
    columns = ['日期', '工程名称', '施工单位', '施工部位', '标号', '方式', '车数',
               '单价', '方量', '出泵费', '空载费', '总价']
    for col_num in range(len(columns)):
        ws.write(row_num, col_num, columns[col_num], font_style)
    # Sheet body, remaining rows
    font_style = xlwt.XFStyle()
    rows = list(queryset)
    for row in rows:
        row_num += 1
        ws.write(row_num, 0, str(row.date), font_style)
        ws.write(row_num, 1, row.project.name, font_style)
        ws.write(row_num, 2, row.project.constructor.name, font_style)
        ws.write(row_num, 3, row.purchase, font_style)
        ws.write(row_num, 4, row.position, font_style)
        ws.write(row_num, 5, row.get_deliver_type_display(), font_style)
        ws.write(row_num, 6, row.num_car, font_style)
        ws.write(row_num, 7, row.unit_price, font_style)
        ws.write(row_num, 8, row.num_cube, font_style)
        ws.write(row_num, 9, row.price_pump, font_style)
        ws.write(row_num, 10, row.price_extra, font_style)
        ws.write(row_num, 11, row.total_price, font_style)

    summary = dict(queryset.aggregate(**metrics))
    row_num += 1
    ws.write(row_num, 6, summary['total_car'], font_style)
    ws.write(row_num, 8, summary['total_cube'], font_style)
    ws.write(row_num, 11, summary['total_price'], font_style)
    wb.save(response)
    return response
```

#### 2. xlsxwriter

##### 整行写入

```python
@login_required()
def export_transaction_xls(request, project_id, date_from, date_to):
    # 读取数据
    date_range = [date_from, date_to]
    queryset = Transaction.objects.day_to_day_archive(date_range).filter(project=project_id)
    project = get_object_or_404(Project, pk=project_id)

    # 写 Excel title
    title = ['日期', '工程名称', '施工单位', '施工部位', '标号', '方式', '车数',
             '单价', '方量', '出泵费', '空载费', '总价']
    filename = "{}-{}.xlsx".format(project.name, f'{date_from}到{date_to}')  # 指定excel文件名
    MEDIA_HOME = settings.MEDIA_DIR
    DOWNLOAD_DIR = os.path.join(MEDIA_HOME, 'excels')  # 指定存放文件的目录
    file_path = "{}/{}".format(DOWNLOAD_DIR, filename)
    workbook = xlsxwriter.Workbook(file_path)  # 这里我用的xlsxwriter导出文件
    table = workbook.add_worksheet(f'{date_from}到{date_to}')

    # Style
    table.set_column('A:A', 14)
    table.set_column('B:B', 20)
    table.set_column('C:C', 25)
    table.set_column('D:D', 25)

    # Add a bold format to use to highlight cells.
    top = workbook.add_format(
        {'font_name': 'Hei', 'font_size': 12, 'bold': True,
         'border': 1, 'align': 'center', 'bg_color': 'cccccc'
         }
    )
    row_format = workbook.add_format({
        'bold': False,
        'font_name': 'Microsoft yahei',
        'font_size': 11,
        'align': 'right',
        'valign': 'vcenter',  # 垂直居中
        # 'fg_color': '#D7E4BC',  # 颜色填充
    })
    merge_format = workbook.add_format({
        'bold': True,
        'font_name': 'Hei',
        'font_size': 12,
        'align': 'right',
        'valign': 'vcenter',  # 垂直居中
        # 'fg_color': '#D7E4BC',  # 颜色填充
    })

    # Add a number format for cells with money.
    money_format = workbook.add_format({'num_format': '$#,##0'})

    # Add an Excel date format.
    date_format = workbook.add_format({'num_format': 'mmmm d yyyy'})
    table.write_row('A1', title, top)

    # 写行
    tmp_line = 2
    for row in queryset:
        # todo 指定每一行的数据
        tmp = [
            f'{row.date.year}年{row.date.month}月{row.date.day}日',
            row.project.name,
            row.project.constructor.name,
            row.position,
            row.purchase,
            row.get_deliver_type_display(),
            row.num_car,
            row.unit_price,
            row.num_cube,
            row.price_pump,
            row.price_extra,
            row.total_price
        ]
        table.write_row("A{}".format(tmp_line), tmp, row_format)  # 自行去查一下write_row方法
        tmp_line += 1

    summary_line = tmp_line - 1
    # 需要汇总的列
    summary_col_dict = {
        'G': 6,
        'I': 8,
        'J': 9,
        'K': 10,
        'L': 11
    }
    # 添加汇总行
    table.merge_range(f'A{tmp_line}:F{tmp_line}', '合计：', merge_format)
    for col, col_id in summary_col_dict.items():
        # table.write(summary_line, col_id, f'=SUM({col}2:{col}{summary_line})', money_format)
        table.write(summary_line, col_id, f'=SUM({col}2:{col}{summary_line})', merge_format)
    workbook.close()

    # 返回串流下载
    response = StreamingHttpResponse(file_iterator(file_path))
    response['Content-Type'] = 'application/octet-stream'
    response['Content-Disposition'] = "attachment;filename={0}".format(
        filename.encode('utf-8').decode('ISO-8859-1')
    )
    return response

```

### 导出 elsx
[参考](https://djangotricks.blogspot.com/2019/02/how-to-export-data-to-xlsx-files.html)