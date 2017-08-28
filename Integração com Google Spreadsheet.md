# Goal
Use a google spreadsheet as a database, with jquery to read it.

## Google Spreadsheet

Create a spreadsheet and go to **Tools > Script editor**. 

```js
// original from: http://mashe.hawksey.info/2014/07/google-sheets-as-a-database-insert-with-apps-script-using-postget-methods-with-ajax-example/

function doGet(e){
  return handleResponse(e);
}

// Usage
//  1. Enter sheet name where data is to be written below
        var SHEET_NAME = "Sheet1";
        
//  2. Run > setup
//
//  3. Publish > Deploy as web app 
//    - enter Project Version name and click 'Save New Version' 
//    - set security level and enable service (most likely execute as 'me' and access 'anyone, even anonymously) 
//
//  4. Copy the 'Current web app URL' and post this in your form/script action 
//
//  5. Insert column names on your destination sheet matching the parameter names of the data you are passing in (exactly matching case)

var SCRIPT_PROP = PropertiesService.getScriptProperties(); // new property service

// If you don't want to expose either GET or POST methods you can comment out the appropriate function

function doPost(e){
  console.log(e);
  return handleResponse(e);
}

function handleResponse(e) {
  // shortly after my original solution Google announced the LockService[1]
  // this prevents concurrent access overwritting data
  // [1] http://googleappsdeveloper.blogspot.co.uk/2011/10/concurrency-and-google-apps-script.html
  // we want a public lock, one that locks for all invocations
  var lock = LockService.getPublicLock();
  lock.waitLock(30000);  // wait 30 seconds before conceding defeat.
  
  try {
    // next set where we write the data - you could write to multiple/alternate destinations
    var doc = SpreadsheetApp.openById(SCRIPT_PROP.getProperty("key"));
    var sheet = doc.getSheetByName(SHEET_NAME);
    
    // we'll assume header is in row 1 but you can override with header_row in GET/POST data
    var headRow = e.parameter.header_row || 1;
    console.log(sheet);
    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow()+1; // get next row
    var row = []; 
    // loop through the header columns
    for (i in headers){
      if (headers[i] == "Timestamp"){ // special case if you include a 'Timestamp' column
        row.push(new Date());
      } else { // else use header name to get data
        row.push(e.parameter[headers[i]]);
      }
    }
    // more efficient to set values as [][] array than individually
    sheet.getRange(nextRow, 1, 1, row.length).setValues([row]);
    // return json success results
    return ContentService
          .createTextOutput(JSON.stringify({"result":"success", "row": nextRow}))
          .setMimeType(ContentService.MimeType.JSON);
  } catch(e){
    // if error return this
    return ContentService
          .createTextOutput(JSON.stringify({"result":"error", "error": e}))
          .setMimeType(ContentService.MimeType.JSON);
  } finally { //release lock
    lock.releaseLock();
  }
}

function setup() {
    var doc = SpreadsheetApp.getActiveSpreadsheet();
    SCRIPT_PROP.setProperty("key", doc.getId());
}
```
Run `Publish > As web app` and `Run > Setup`. Go back to your spreadsheet and publish it in **File > Publish**.

## Javascript

I'm sorry there's so much going on here but it's too valuable to cut it off. I'll try to explain, tho.

I was builting a raffle application, where people got more chances to winning when share a special link with a friend. To doing so, I've used google's api to shorten links; an ecript-decript function to scramble the e-mail variable (witch comes with url) and my spreadsheet. One should register only once, so I had to read the table to see if their e-mail was there.

When one register with a referee, a hidden field carries the friend's email and by doing so, this person got in the table twice, getting more chances to win.

The raffle was cancelled but the coding was working. [You can see the layout in my Behance](https://www.behance.net/gallery/56046447/Landing-page).

In the script below, changeable values are between `[]`.

### Scripts required
* jQuery
* [jQuery URL Shortener 1.0](https://github.com/hayageek/jQuery-URL-shortener)
* jQuery Validate


```js
//**************************************/
// Get all registered e-mails so there's no duplicate
var prohibitedEmails = [];
$.getJSON("[SPREADSHEET PUBLIC URL]", function(data) {
    $(data.feed.entry).each(function(i, e){
        var item = {};
        item["email"] = e.title.$t;
        prohibitedEmails.push(item);
    })
  });

//**************************************/
$(document).ready(function(){
    // Smooth scroll
    smoothScroll();

    $.validate({
        lang: 'pt',
        modules : 'html5'
      });
    
    $.urlShortener.settings.apiKey='[GOOGLE API KEY]';
    
    if($("#thankyou").length == 0) {
        // Is not Thank You page

        // catch email
        $("#cadastro").focusout(function(){
            var email = $(this).val();
            
            if(email.length > 0){
                var listSearch = search(prohibitedEmails,email);
                if (listSearch.length > 0){
                    $("#jacadastrado").show();
                    $("#cadastro").addClass("error");
                } else { 
                    $("#jacadastrado").hide();
                    $("#cadastro").removeClass("error");
                    // Only register if there's no e-mail like this on the spreadsheet
                    var cadastroadress = $("#cadastro").val();
                    var sharelinkurl = "[RAFFLE ADRESS]/index.html?ind="+utoa(cadastroadress);
                    
                    $.urlShortener({
                        longUrl: sharelinkurl,
                        success: function (shortUrl) {
                            $("#sharelink").val(shortUrl);
                            localStorage.setItem("sharelinkurl", shortUrl);
                        },
                        error: function(err)
                        {
                            console.log(JSON.stringify(err));
                        }
                    });
        
                    localStorage.setItem("cadastroadress", cadastroadress);
                }
            }
        });    

        // When select "other" in select box add an text input.
        $("#area").change(function(){
            var area = $(this).val();
            if(area == "Outros"){
                $(this).attr("name","");
                $(this).parent().addClass("half");
                $("#otherarea").show().addClass("half").append("<input type='text' name='area' id='area' placeholder='Qual?' required data-validation='required'>")
            } else {
                $(this).attr("name","area");
                $(this).parent().removeClass("half");
                $("#otherarea").empty().hide();
            }
        })

        //**************************************/   
        // Fetch indicator from URL
        var indicationURL = window.location.href;
        
        var params = {};

        if (indicationURL) {
            var parts = window.location.search.substring(1).split('&');

            for (var i = 0; i < parts.length; i++) {
                var nv = parts[i].split('=');
                if (!nv[0]) continue;
                params[nv[0]] = nv[1] || true;
            }
        }

        var indicator = params.ind;

        if (indicator != null){
            $("#indicacao").val(atou(indicator));
        }

        //**************************************/   
        // Submit form
        $("form").submit(function(e){
            e.preventDefault();

            var email = $("#cadastro").val();
            
            var listSearch = search(prohibitedEmails,email);
            if (listSearch.length === 0){
                var $form = $(this);
                var serializedData = $form.serialize();
                var url = "[GOOGLE SCRIPT URL]";
                
                var jqxhr = $.post(url, serializedData, function(data, textStatus) {
                    window.location.href = "[RAFFLE ADRESS]/obrigado.html";
                }, "json");
            } else {
                $("#jacadastrado").show();
                $("#cadastro").addClass("error");
            }
            return false;
        });

    } else {
        // Thank you page
        sharelinkurl = localStorage.getItem("sharelinkurl");
        $("#linkBox").text(sharelinkurl);       

    }

}); // end document.ready

//**************************************/  
// Encript and decript

// utoa('✓ à la mode'); // 4pyTIMOgIGxhIG1vZGU=
function utoa(str) {
    return window.btoa(unescape(encodeURIComponent(str)));
}
// atou('4pyTIMOgIGxhIG1vZGU='); // "✓ à la mode"
function atou(str) {
    return decodeURIComponent(escape(window.atob(str)));
}

//**************************************/
function smoothScroll(){
    $('a[href*="#"]')
        .not('[href="#"]')
        .not('[href="#0"]')
        .click(function(event) {
            if (
                location.pathname.replace(/^\//, '') == this.pathname.replace(/^\//, '') 
                && 
                location.hostname == this.hostname
            ) {
                var target = $(this.hash);
                target = target.length ? target : $('[name=' + this.hash.slice(1) + ']');
                if (target.length) {
                event.preventDefault();
                $('html, body').animate({
                    scrollTop: target.offset().top
                }, 1000, function() {
                    var $target = $(target);
                    $target.focus();
                    if ($target.is(":focus")) { // Checking if the target was focused
                        return false;
                    } else {
                        $target.attr('tabindex','-1'); // Adding tabindex for elements not focusable
                        $target.focus(); // Set focus again
                    };
                });
            }
        }
    });
}


function search(source, name) {
    var results;
    
    name = name.toUpperCase();
    results = $.map(source, function(entry) {
        var match = entry.email.toUpperCase().indexOf(name) !== -1;
        return match ? entry : null;
    });
    return results;
}
```

## HTML

No big deal here, just a simple form.

```html
<form action="#" onsubmit="return false;" id="registration">
        <ul class="cleanList">
          <li class="half">
            <label for="name">Nome</label>
            <input type="text" name="name" id="name" placeholder="Nome" required="" data-validation="required"
            data-validation-error-msg="Insira seu nome">
          </li>
          <li class="half">
            <label for="lastname">Sobrenome</label>
            <input type="text" name="lastname" id="lastname" placeholder="Sobrenome" required=""
            data-validation="required" data-validation-error-msg="Insira seu sobrenome">
          </li>
          <li>
            <label for="phone">Telefone com DDD</label>
            <input type="number" id="phone" name="phone" placeholder="Telefone com DDD" required=""
            data-validation="number length" data-validation-length="10-11">
          </li>
          <li>
            <label for="cadastro">E-mail para cadastro</label>
            <input type="email" id="cadastro" name="cadastro" placeholder="E-mail para cadastro"
            required="" data-validation="email">
          </li>
          <li>
            <span id="jacadastrado" class="form-error help-block" style="display:none">E-mail já cadastrado!</span>
          </li>
          <li>
            <label for="site">Site ou FB da empresa</label>
            <input type="text" id="site" name="site" placeholder="Site ou FB da empresa" required=""
            data-validation="required" data-validation-error-msg="Forneça seu site ou página do FB">
          </li>
          <li id="atuacao">
            <label for="area">Área de atuação</label>
            <select name="area" id="area" required="" data-validation="required">
              <option value="">Área de atuação</option>
              ...
              <option value="Outros">Outros</option>
            </select>
          </li>
          <li id="otherarea"></li>
          <input type="hidden" id="indicacao" name="indicacao">
          <input type="hidden" id="sharelink" name="sharelink">
          <li class="tac">
            <button type="submit" class="button purpleBg">Inscreva-se</button>
          </li>
        </ul>
      </form>
```

Hope it helps.
