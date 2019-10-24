
> 将格式改成如下形式：

一 、
```
jQuery(document).ready(function(){
    jQuery(function () {
    //code
});
```

二、
```
jQuery(document).ready(function($){
 
    $(function () {
  	//code
});
```

三、
```
(function($,undefined){ 
   $(document).ready(function(){ 
       //code
   }); 
})(jQuery);
```

我用第三种方法，因为之前都是这样的：
```
(function ($) {
    $(document).ready(function () {
        console.log("It's here")
        let col_unit_price = ".field-unit_price input";
        let col_num_cube = ".field-num_cube input";
        let col_other_cost = ".field-other_cost input";
        let col_total_price = ".field-total_price input";

        // $(col_total_price).prop('disabled', true);

        $(".field-unit_price input, .field-num_cube input, .field-other_cost input").on('keyup', function () {

            let unit_price = $(col_unit_price).val() ? $(col_unit_price).val() : 0;
            let num_cube = $(col_num_cube).val() ? $(col_num_cube).val() : 0;
            let other_cost = $(col_other_cost).val() ? $(col_other_cost).val() : 0;
            let total_price = Number(unit_price) * Number(num_cube) + Number(other_cost);
            $(col_total_price).attr('disabled', true);
            $(col_total_price).val(total_price);
        });


        $("#transaction_form").submit(function (event) {
            $("input, select").removeAttr('disabled');
        });
    });
}(django.jQuery));
```
所以直接去掉末尾的 `django`就可以了