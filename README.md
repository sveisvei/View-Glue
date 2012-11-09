     _   _                               ___    _                 
    ( ) ( ) _                           (  _`\ (_ )               
    | | | |(_)   __   _   _   _  ______ | ( (_) | |  _   _    __  
    | | | || | /'__`\( ) ( ) ( )(______)| |___  | | ( ) ( ) /'__`\
    | \_/ || |(  ___/| \_/ \_/ |        | (_, ) | | | (_) |(  ___/
    `\___/'(_)`\____)`\___x___/'        (____/'(___)`\___/'`\____)
                                                                                                                                

# View Glue

Helps you with event binding and gives you a possibility to lazyload, so you can worry about something else.

View glue uses jquery for event delegation, require js to run/load modules and glues it together.
(Target: I want this to use a more lightweight loader, supported by a module-server for a small client-footprint.)

# The why?

Having actions that doesnt always have its module/javascript-code loaded should be painless to load runtime.
If the code is loaded, it will run right away.

# Event binding

It will replace:
    
```javascript    
function bindEvents(){
  $(".someselector").on('submit',           somemodule.submit)
  $(".someselector").on('click', 'element'  somemodule.clickElement)
  $(".someselector").on('click',            somemodule.click)
  $(".someselector").on('focus',            somemodule.focus)
  $(".someselector").on('touchstart',       somemodule.click)
  bindEventsModule2();
}
function bindEventsModule2(){
  $(".someselector2").on('submit',           somemodule.submit)
  $(".someselector2").on('click', 'element'  somemodule.clickElement)
  $(".someselector2").on('click',            somemodule.click)
  $(".someselector2").on('focus',            somemodule.focus)
  $(".someselector2").on('touchstart',       somemodule.click)
}
$(document).ready(bindEvents);
```

With:
```javascript
//VOID JS, except global event delegation:
viewGlue.startContext(...)
```    

But instead of setting up js, you will have to mark-it-up:
    - <a href="/some/real/url" data-r="somemodule.show"></a>


And force you to serve initial view, and not do magic on init. There is no init *waves hand*



# Getting started

Creating an instance:

```javascript
var viewGlue = new ViewGlue({
    loader: window.require
});
```    

Starting to delegate events:

```javascript
var $root         = $(".page");
var lazySelectors = {$content: '.content'};
var defaults      = {hello: 'Hello world'};
// Start:
var glueInstance  = viewGlue.startContext('contextName', $root, 'all', lazySelectors,  defaults);
```

viewGlue.startContext Arguments:

* 1 String ) context creates an instance, multiple instances can run on same page, but not on same $root element, e.g. multiple widgets on same page
* 2 Jquery-collection ) Root-element ($root)
* 3 String, kommasep ) Event-tags('all', or 'forms,input,history,touch,normal...') to be delegated
* 4 Object ) Selectors to be lazyloaded, and available inside module
* 5 Object ) defaults to be loaded into module


#Example 1

Markup:

```html
    <!--     1) load module with modulename -->
    <a href="/some/url" data-r="context:modulename.methodname">Link</a>
```    

Module "modulename"

```javascript
define([], function(){
    function methodname(module){
        // 3) call with selectors and defaults from context, defaults to delegated $root.
        module.$root.append('<p>'+module.hello+'!!!</p>');
    }
    return {
        // 2) will call methodname
        methodname: methodname
    };
});
```



#Example 2

Markup:

```html
<!--     1) load module with modulename on click -->
<div class="content">
    <button data-r="modulename2">Render module</button>
</div>
```

Module "modulename2"

```javascript
define([], function(){
    function renderView(module){
        // 3) call run/renderView with selectors and defaults from context, defaults to delegated $root.
        module.$content().append('<p> Module: '+module.name+'</p>');
    }
    function methodname(module){
        module.$root.append('<p>'+module.hello+'!!!</p>');
    }
    return {
        config: {
            // 2) will call modules config.run function
            run: renderView
        }
        methodname: methodname
    };
});
```    




#Example 3
Markup:

```html
<!--     1) load module with modulename on submit or focus on input-->
<form data-r-submit="modulename.submit">
    <input type="email" name="email" data-r-focus="modulename.validate" />
</form>
```

Module "modulename2"

```javascript
define([], function(){
    function successHandler(module){
        return function(data){
            var message = '<p>Takk for at du sendte inn skjema. Bekreftelse er sendt til ' + module.input.email + '</p>';
            // module.$elem is the element that the data-r attributt resides on.
            module.$elem.replaceWith(message);
        }
    }
    /*
        3) call with selectors and defaults from context, defaults to delegated $root.
           Forms usually needs to handle input, so the ViewGlue 
           collects the whole <form> into module.input object{name: val()} for easy access
    */
    function submit(module){
        $.ajax('/remote', module.input).success(successHandler(module));
    }
    function validate(module){
        module.$elem.one('blur', function(){
           var val = module.$elem.val();
           if (val == ''){
              $('<p>Error,  field is required</p>').insertAfter(module.$elem);
           }
        });
    }
    return {
        // 2) will submit or validate, depending on method
        submit: submit
        validate: validate
    };
});
```

#Example 4

Markup:

```html
<!--
1)  "Root" handler collects from children when clicked,
    event-bubling goes up and collectes a valid target(data attributes,
    except validMap option) and a valid action(data-r or alike)
-->
<ul data-r="modulename3.action">
    <li data-id="1">
        Elem 1
        <span>(some element)</span>
    </li>
    <li data-id="2">
        Elem 2
        <span>(some element)</span>
    </li>
    <li data-r="modulename3.reset">
    </li>
</ul>
```


Module "modulename3"

```javascript
define([], function(){
    function action(module){
        // target is the element that is clicked on, but not the <span> because it doesnt have any "valid" data-attributes
        module.$target.addClass('active').siblings().removeClass('active');
        // the target data is collected, and extends the data from the $elem
        module.$elem.find('[data-r="modulename3.reset"]').text('Selected '+module.id);
    }
    function remove(module){
        module.$elem.html('');
    }
    return {
        action: action,
        reset: reset
    };
});
```

    


