---
description: Some notes on using the scala play framework to build a web app
tags: scala play web app
---

I recently started to use scala and wanted to learn to use its powerful framework for web application : Play.

In this small write up, i'll present what I was able to build with relatively basic features. This won't be about the raw basic but more about my discoveries.

## Rendering a templated HTML page 

The first thing that appeared me attractive was the template engine. It allows to have scala code in the html view page that you use.



A scala.html file has the following format
```html
@() <!-- function definition -->

<!-- html code -->
```

The adavantage of this format is the ability to add parameters to the html when the page is called. Example :

```html
@(title: String) <!-- function definition -->

<h1> @title </h1>

```
This code will create a page with a title defined by the given parameter in input


Another advantage of this format is the usage of a common format for all pages. The file `app/views/main.scala.html` demonstrates it.

```html
@(title: String)(content: Html)
```

The parameters are the title of the webpage and the content of the inner page. Have a look at the screenshot of the following page which uses this simple code.

```html
@()
@main("Dummy"){
  <h1> Ok </h1>
}
```

![Example generated with main usage](https://raw.githubusercontent.com/AdMoR/social-partner-website/master/doc/img/basic_main.png)

The main script handles the header, menu and footer. You just have to code your content inside the page.


## Creating a form formatted on a case class

Another interesting feature from play is the form integrating to create a specific class. Have a look at the following pieces of code.

```scala
// Our case representing the data we will create with our form
case class Event(id: Long, title: String, description: String, groupName: String, tags: List[String], eventType: String)


@javax.inject.Singleton
class EventController @javax.inject.Inject() (override val app: Application, messagesAction: MessagesActionBuilder)
  extends AuthController("home") with play.api.i18n.I18nSupport {
  ApplicationDatabase.migrateSafe()

  // This is more or less the mockup for our database
  var events = Map[Long, Event](0.toLong -> Event(0.toLong, "My first event", "Cool stuff", "Amicale scolaire",
    List("Entraide"), "Entraide"), 1.toLong -> Event(1, "My first event", "Cool stuff", "Amicale scolaire", List("Entraide"), "Entraide"))
  var eventCounter = 2.toLong

  // The form here defines what each will should have 
  val eventForm = Form(
    mapping(
      "id" -> ignored(eventCounter.toLong),
      "title" -> nonEmptyText,
      "description" -> nonEmptyText,
      "groupName" -> text,
      "tags" -> list(text),
      "eventType" -> text
    )(Event.apply)(Event.unapply)
  )

  def listing(userId: String) = withoutSession("") { implicit request => implicit td =>
    Future.successful(Ok(views.html.category(List(events { userId.toLong }))))
  }

  def form() = withoutSession("") { implicit request => implicit td =>
    eventCounter += 1
    Future.successful(Ok(views.html.form(eventForm)))
  }

  def submit = Action { implicit request =>
    val event = eventForm.bindFromRequest.get
    events = events + (eventCounter -> event)
    Redirect(s"event/${eventCounter.toString}")
  }

}

```

This is a lot of code but I will break it down

```scala
case class Event(id: Long, title: String, description: String, groupName: String, tags: List[String], eventType: String)
```
Our data class

```scala
  val eventForm = Form(
    mapping(
      "id" -> ignored(eventCounter.toLong),
      "title" -> nonEmptyText,
      "description" -> nonEmptyText,
      "groupName" -> text,
      "tags" -> list(text),
      "eventType" -> text
    )(Event.apply)(Event.unapply)
  )
```
The form used for the mapping from the HTML form to the class

```scala
  def form() = withoutSession("") { implicit request => implicit td =>
    eventCounter += 1
    Future.successful(Ok(views.html.form(eventForm)))
  }
```
We access the form html page through this function and get the following :
![The HTML form]({{site.baseurl}}/assets/images/scala_form.png)

Once the field are filled and sent to the server, the following part is called : 

```scala
  def submit = Action { implicit request =>
    val event = eventForm.bindFromRequest.get
    events = events + (eventCounter -> event)
    Redirect(s"event/${eventCounter.toString}")
  }
```

the `event` object is an Event built from the fields retrieved. It can then be added to the pseudo database (here a list). And we can then render the page for this event.


