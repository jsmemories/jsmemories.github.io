---
layout: post
title: CREATING AND DEFINING BASE VIEWS FOR BACKBONE.JS
description: "CREATING AND DEFINING BASE VIEWS FOR BACKBONE.JS"
author: asgaroth
modified: 2013-10-21
tags: [backbone, views]

comments: true
share: true
---

One thing I always advice to do when working on backbone applications, regardless of the application size, is to create a BaseView that extends from Backbone.View. It will help you add some fancy functionality that will be shared across your other views, and allows you to keep it DRY.

For example, one thing I love to always add, is the ability to specify the name of a template in a view, and not worry about how they are compiled, or what templating engine it will use underneath, or where is it gonna pull its contents:


{% highlight javascript %}
var profile = new BaseView({
    templateName: 'user_profile'
});
{% endhighlight %}


BaseView can take the template contents either from a variable containing a precompiled template, or fetch the HTML of a script tag and compile it using any template engine without our view having to know anything about that.

There is a nice post by Ian Storm Taylor, about [extending configuration logic in backbone](http://ianstormtaylor.com/backbonejs-configuration-logic-should-be-extendable/) I suggest reading that post, he elaborates on some options for extending the Backbone.Model configuration. today we will work with his idea of [augmenting the constructor](http://ianstormtaylor.com/backbonejs-configuration-logic-should-be-extendable/#augment_the_constructor_incomplete), in order to build our BaseView.

CREATING THE BASEVIEW

Let’s get our hands dirty and create our BaseView, I am using simple vars, but in a real world you should be using namespaces

{% highlight javascript %}
// Don't do this at home, use namespaces!!!
var BaseView = Backbone.View.extend({
    // The name of the template to use for look-up.
    templateName: null,
    constructor: function() {
        this.configure.apply(this, arguments);
        Backbone.View.apply(this, arguments);
    },
    configure: function(options){
        if (options.templateName) {
            this.template = Handlebars.compile(
                $('#'+options.templateName).html()
            );
        }
    },
});
{% endhighlight %}

When instantiating a view of this type, the first thing it will do, is call the configure method, it will check for an element with id equal to the *templateName* attribute, and initialize our *template*

So for example,  we could have the template defined like this:

{% highlight javascript %}
<script id="user" type="text/x-handlebars-template">
 {{name}}: age {{age}}
</script>
{% endhighlight %}

And then we can just define a view that uses the template as simple as this:

{% highlight javascript %}
var userView = new BaseView({
    templateName: 'user',
    initialize: function(){
        this.render();
    },
    render: function(){
        this.$el.html(this.template({
            name: "John",
            age: 25
        }));
    }
});
{% endhighlight %}

That is very nice, but the code on *render* its very repetitive in Backbone applications, now that we have our base view we can defined something that would help us get rid of that.

Normally what happens in the *render* is that we serialize our models/collections, call the template, and replace the html.

{% highlight javascript %}
render: function(){
    // does this look familiar?
    this.$el.html(this.template(this.model.toJSON()));
 
    // how about this?
    this.$el.html(this.template({
        items: this.collection.toJSON()
    }));
 
    return this;
}
{% endhighlight %}

Now that we have our BaseView, we can get rid of that repetitive code, lets simply define a default render that does this same functionality, but move the logic of serializing the data, so that we make it flexible and overridable but our child views:

{% highlight javascript %}
_serializeData: function(){
   var data = {};
 
   if (this.model) {
     data = this.model.toJSON();
   }
   else if (this.collection) {
     data.items = this.collection.toJSON();
   }
 
   return data;
},
 
serializeData: function(){
    return this._serializeData();
},
 
render: function(){
    var data = this.serializeData();
    this.$el.html(this.template(data));
    this.trigger("afterRender");
    return this;
}
{% endhighlight %}

That look very nice, render calls serializeData, which by default serializes our model/collection, and now render also triggers an event! which comes in handy sometimes.

Now, not only we can override serializeData in our views if we need to add some more variables, but can also call *this._serializeData();* within it, and have the model/collection serialize, giving us even more flexibility.

{% highlight javascript %}
// in a child view:
var userView = new BaseView({
templateName: 'user',
    initialize: function(){
        this.render();
    },
    serializeData: function(){
        var data = {
            model: this._serializeData()
        };
 
        data.blog = "JsMemories";
 
        var option = this.model.get('age');
        data.ageLabel = (age < 10 ? "Sorry you are too young" : "Welcome!";
        return data;
    }
});
{% endhighlight %}

ADDING ADDITIONAL VARIABLES

But wait… why stop there? that passing of additional variables looks like something we can improve, we have our BaseView and we can add some more magic to it, how about we let out child views define those additional variables that we need for the template, in a more declarative way?

Being able to defined our view like this before would have been nice:

{% highlight javascript %}
var userView = new BaseView({
    templateName: 'user',
    templateVars: {
        blog: "JsMemories",
        ageLabel: function(){
            return  this.model.get('age') age < 10 ?
                     "Sorry you are too young" :
                     "Welcome!";
        }
    },
    initialize: function(){
        this.render();
    }
});
{% endhighlight %}

Adding that functionality is as simple as modifying our *_serializeData* in BaseView:

{% highlight javascript %}
_serializeData: function(){
   var data = {};
 
   if (this.model) {
     data.model = this.model.toJSON();
   }
   else if (this.collection) {
     data.items = this.collection.toJSON();
   }
 
   if (this.templateVars) {
        var self = this;
        var obj = _.clone(this.templateVars);
        obj = _.object(_.map(obj, function(val, key){
            if (_.isFunction(val)) {
                val = val.call(self);
            }
            return [key, val];
        }));
        _.extend(data, obj);
   }
 
   return data;
}
{% endhighlight %}

You can add many fancy things like this to your BaseView, as you can see its pretty easy, and it helps you keep the rest of the Views in your application very simple and concrete. I hope this help you in your applications regardless of the size, to be more organized and DRY.

This is the complete BaseView from our exercise:

{% highlight javascript %}
// Don't do this at home, use namespaces!!!
var BaseView = Backbone.View.extend({
    // The name of the template to use for look-up.
    templateName: null,
    constructor: function() {
        this.configure.apply(this, arguments);
        Backbone.View.apply(this, arguments);
    },
    configure: function(options){
        // You can add your code here to take the template
        // from a global cache if you want
        // ie: return App.TEMPLATES[options.templateName]
        if (options.templateName) {
            this.template = Handlebars.compile(
                $('#'+options.templateName).html()
            );
        }
    },
 
    _serializeData: function(){
       var data = {};
 
       if (this.model) {
         data.model = this.model.toJSON();
       }
       else if (this.collection) {
         data.items = this.collection.toJSON();
       }
 
       if (this.templateVars) {
            var self = this;
            var obj = _.clone(this.templateVars);
            obj = _.object(_.map(obj, function(val, key){
                if (_.isFunction(val)) {
                    val = val.call(self);
                }
                return [key, val];
            }));
            _.extend(data, obj);
       }
 
       return data;
    }
 
    serializeData: function(){
        return this._serializeData();
    },
 
    render: function(){
        var data = this.serializeData();
        this.$el.html(this.template(data));
        this.trigger("afterRender");
        return this;
    }
});
{% endhighlight %}

This is just my way to approach things, I hope to see what other things you would add to a BaseView, and if you were already creating BaseViews what kind of functionality you add there?