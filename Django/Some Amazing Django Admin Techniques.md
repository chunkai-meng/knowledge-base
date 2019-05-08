# Some Amazing Django Admin Techniques

1. To show a beautiful percentage chart in *List Display* & *Change Page*

![Change List](Some Amazing Django Admin Techniques.assets/eDoUk.jpg)



![change](Some Amazing Django Admin Techniques.assets/image-20190504190600785.png)

Model

```python
# models.py
    
class SomeModel(models.Model):
    ....
    def percentage_paid(self):
        if self.paid_amount and self.final_price:
            percentage = round((self.paid_amount / self.final_price * 100), 2)
        else:
            percentage = 0
        return format_html(
            '''
            <progress value="{0}" max="100"></progress>
            <span style="font-weight:bold">{0}%</span>
            ''',
            percentage
        )
```

Admin

```python
# admin.py

class CaseAdmin(CommonInfoAdmin):
    fieldsets = CommonInfoAdmin.fieldsets + (
        ('Sales Summary', {
            'fields': (
                ....
                ('final_price', 'paid_amount', 'balance', 'percentage_paid'),
                ....
            )
        }),
    )
    readonly_fields = [
        ....
        'percentage_paid',
    ]
    list_display = (
        ....
        'percentage_paid'
    )
```
