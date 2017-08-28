# indyGuideSkill
var Alexa = require('alexa-sdk');
var http = require('http');

var states = {
    SEARCHMODE: '_SEARCHMODE',
    TOPFIVE: '_TOPFIVE',
};

var location = "Indianapolis";

var numberOfResults = 3;

var APIKey = "c7da21dda76f4d8c896c2dd5ff7d82eb";

var welcomeMessage = location + " Guide. You can ask me for an attraction, the local news, or  say help. What will it be?";

var welcomeRepromt = "You can ask me for an attraction, the local news, or  say help. What will it be?";

var locationOverview = "Indianapolis is the capital of Indiana located in the central part of the state. As of 2016, it had an estimated population of 855,164, making it the largest city in Indiana and the second largest in the Midwest. What else would you like to know?";

var HelpMessage = "Here are some things you can say: Give me an attraction. Tell me about " + location + ". Tell me the top five things to do. Tell me the local news.  What would you like to do?";

var moreInformation = "See your  Alexa app for  more  information."

var tryAgainMessage = "please try again."

var noAttractionErrorMessage = "There was an error finding this attraction, " + tryAgainMessage;

var topFiveMoreInfo = " You can tell me a number for more information. For example open number one.";

var getMoreInfoRepromtMessage = "What number attraction would you like to hear about?";

var getMoreInfoMessage = "OK, " + getMoreInfoRepromtMessage;

var goodbyeMessage = "OK, have a nice time in " + location + ".";

var newsIntroMessage = "These are the " + numberOfResults + " most recent " + location + " headlines, you can read more on your Alexa app. ";

var hearMoreMessage = "Would you like to hear about another top thing that you can do in " + location +"?";

var newline = "\n";

var output = "";

var alexa;

var attractions = [
    { name: "Indianapolis Museum of Art", content: "The IMA is located just northwest of downtown Indianapolis. The 152 acre campus holds the 9th oldest art museum in the US, as well as many outdoor sculptures and gardens.", location: "4000 Michigan Rd, Indianapolis, IN", contact: "info@imamuseum.org\n 317 923 1331" },
    { name: "White River State Park", content: "Located on the edge of downtown, White River State Park contains 250 acres of popular Indy attractions, including the Central Canal, White River Gardens, and Military Park. Guests can rent bikes, pedal boats, and kayaks to better explore the urban state park.", location: "801 W Washington St, Indianapolis, Indiana", contact: "317 233 2434" },
    { name: "The Children's Museum of Indianapolis", content: "Indianapolis is home to the world's largest children's museum. Full of exciting games and seasonal exhibits, this attraction is fun for the whole family.", location: "3000 N Meridian St, Indianapolis, IN", contact: "317 334 4000" },
    { name: "Soldiers' and Sailors' Monument", content: "Located in the epicenter of the city, the Soldiers' and Sailors' Monument was built in the late 19th century to honor Hoosiers who fought in the American Civil and Revolutionary Wars. The monument is built in the middle of Circle Center, which is featured on the Indianapolis city flag.", location: "1 Monument Cir, Indianapolis, IN", contact: "317 232 7615" },
    { name: "Indianapolis Zoo", content: "The Indianapolis Zoo contains 64 acres of your typical zoo animals, such as lions, tigers, and polar bears, as well as some more unique attractions, including a dolphin show and a 3 acre botanical garden.", location: "1200 W Washington St, Indianapolis, IN", contact: "317 630 2001" },
];

var topFive = [
    { number: "1", caption: "Walk alongside the Indiana Central Canal, or even rent a paddle boat to take down the Canal.", more: "The Indiana Central Canal was originally dug in the early 1800s. Now, the 3 mile Canal Walk is a popular walking and biking path, with many restaurants and shops to visit along the way.", location: "801 W Washington St, Indianapolis, IN", contact: "317 233 2434" },
    { number: "2", caption: "Spend an afternoon shopping and eating in the popular neighborhood of Broad Ripple.", more: "One of seven cultural districts in Indianapolis, Broad Ripple features many unique eateries and shops. Located just six miles north of Downtown, it's easily accessable via the Monon Bike Trail", location: "Broad Ripple, Indianapolis, IN", contact: "info@brva.org \n 317 251 2782" },
    { number: "3", caption: "Sample any number of delicious ethnic foods or buy a locally made gift for a friend at the Indianapolis City Market", more: "Located just east of Monument Circle, Indianapolis City Market is a shopping center full of locally-made jewelry, trinkets, and gifts, as well as foods from many differents countries and cultures.", location: "222 E Market St, Indianapolis, IN", contact: "317 634 9266" },
    { number: "4", caption: "Go on a tour of the Indianapolis Motor Speedway, home of the Indy 500.", more: "The Indianapolis Motor Speedway opened in 1909 and has hosted many races, shows, and concerts since. The Indianapolis Motor Speedway Museum, recognized as a National Historic Landmark, is one of the most recognized auto racing museums in the world, and offers tours of the track by bus.", location: "4790 W 16th St, Indianapolis, IN 46222", contact: "317 492 8500" },
    { number: "5", caption: "Catch an Indians game at Victory Field.", more: "Indianapolis' own minor league baseball team has many home games throughout the summer, as well as many themed entry days. Victory Field was called the Best Minor League Ballpark in America by both Sports Illustrated and Baseball America.", location: "501 W Maryland St, Indianapolis, IN 46225", contact: "317 269 3545" }
];

var topFiveIntro = "Here are the top five things to  do in " + location + ".";

var newSessionHandlers = {
    'LaunchRequest': function () {
        this.handler.state = states.SEARCHMODE;
        output = welcomeMessage;
        this.emit(':ask', output, welcomeRepromt);
    },
    'getAttractionIntent': function () {
        this.handler.state = states.SEARCHMODE;
        this.emitWithState('getAttractionIntent');
    },
    'getTopFiveIntent': function(){
        this.handler.state = states.SEARCHMODE;
        this.emitWithState('getTopFiveIntent');
    },
    'AMAZON.StopIntent': function () {
        this.emit(':tell', goodbyeMessage);
    },
    'AMAZON.CancelIntent': function () {
        // Use this function to clear up and save any data needed between sessions
        this.emit(":tell", goodbyeMessage);
    },
    'SessionEndedRequest': function () {
        // Use this function to clear up and save any data needed between sessions
        this.emit('AMAZON.StopIntent');
    },
    'Unhandled': function () {
        output = HelpMessage;
        this.emit(':ask', output, welcomeRepromt);
    },
};

var startSearchHandlers = Alexa.CreateStateHandler(states.SEARCHMODE, {
    'getOverview': function () {
        output = locationOverview;
        this.emit(':askWithCard', output, location, locationOverview);
    },
    'getAttractionIntent': function () {
        var cardTitle = location;
        var cardContent = "";

        var attraction = attractions[Math.floor(Math.random() * attractions.length)];
        if (attraction) {
            output = attraction.name + " " + attraction.content + newline + moreInformation;
            cardTitle = attraction.name;
            cardContent = attraction.content + newline + attraction.contact;

            this.emit(':tellWithCard', output, cardTitle, cardContent);
        } else {
            this.emit(':ask', noAttractionErrorMessage, tryAgainMessage);
        }
    },
    'getTopFiveIntent': function () {
        output = topFiveIntro;
        var cardTitle = "Top Five Things To See in " + location;

        for (var counter = topFive.length - 1; counter >= 0; counter--) {
            output += " Number " + topFive[counter].number + ": " + topFive[counter].caption + newline;
        }
        output += topFiveMoreInfo;
        this.handler.state = states.TOPFIVE;
        this.emit(':askWithCard', output, topFiveMoreInfo, cardTitle, output);
    },
    'AMAZON.YesIntent': function () {
        output = HelpMessage;
        this.emit(':ask', output, HelpMessage);
    },
    'AMAZON.NoIntent': function () {
        output = HelpMessage;
        this.emit(':ask', HelpMessage, HelpMessage);
    },
    'AMAZON.StopIntent': function () {
        this.emit(':tell', goodbyeMessage);
    },
    'AMAZON.HelpIntent': function () {
        output = HelpMessage;
        this.emit(':ask', output, HelpMessage);
    },
    'getNewsIntent': function () {
        httpGet(location, function (response) {

            // Parse the response into a JSON object ready to be formatted.
            var responseData = JSON.parse(response);
            var cardContent = "Data provided by New York Times\n\n";

            // Check if we have correct data, If not create an error speech out to try again.
            if (responseData == null) {
                output = "There was a problem with getting data please try again";
            }
            else {
                output = newsIntroMessage;

                // If we have data.
                for (var i = 0; i < responseData.response.docs.length; i++) {

                    if (i < numberOfResults) {
                        // Get the name and description JSON structure.
                        var headline = responseData.response.docs[i].headline.main;
                        var index = i + 1;

                        output += " Headline " + index + ": " + headline + ";";

                        cardContent += " Headline " + index + ".\n";
                        cardContent += headline + ".\n\n";
                    }
                }

                output += " See your Alexa app for more information.";
            }

            var cardTitle = location + " News";

            alexa.emit(':tellWithCard', output, cardTitle, cardContent);
        });
    },

    'AMAZON.RepeatIntent': function () {
        this.emit(':ask', output, HelpMessage);
    },
    'AMAZON.CancelIntent': function () {
        // Use this function to clear up and save any data needed between sessions
        this.emit(":tell", goodbyeMessage);
    },
    'SessionEndedRequest': function () {
        // Use this function to clear up and save any data needed between sessions
        this.emit('AMAZON.StopIntent');
    },
    'Unhandled': function () {
        output = HelpMessage;
        this.emit(':ask', output, welcomeRepromt);
    }
});

var topFiveHandlers = Alexa.CreateStateHandler(states.TOPFIVE, {
    'getAttractionIntent': function () {
        this.handler.state = states.SEARCHMODE;
        this.emitWithState('getAttractionIntent');
    },
    'getOverview': function () {
        this.handler.state = states.SEARCHMODE;
        this.emitWithState('getOverview');
    },
    'getTopFiveIntent': function () {
        this.handler.state = states.SEARCHMODE;
        this.emitWithState('getTopFiveIntent');
    },
    'AMAZON.HelpIntent': function () {
        output = HelpMessage;
        this.emit(':ask', output, HelpMessage);
    },

    'getMoreInfoIntent': function () {
        var slotValue = this.event.request.intent.slots.attraction.value;
        var index = parseInt(slotValue) - 1;

        var selectedAttraction = topFive[index];
        if (selectedAttraction) {

            output = selectedAttraction.caption + ". " + selectedAttraction.more + ". " + hearMoreMessage;
            var cardTitle = selectedAttraction.name;
            var cardContent = selectedAttraction.caption + newline + newline + selectedAttraction.more + newline + newline + selectedAttraction.location + newline + newline + selectedAttraction.contact;

            this.emit(':askWithCard', output, hearMoreMessage, cardTitle, cardContent);
        } else {
            this.emit(':ask', noAttractionErrorMessage);
        }
    },

    'AMAZON.YesIntent': function () {
        output = getMoreInfoMessage;
        alexa.emit(':ask', output, getMoreInfoRepromtMessage);
    },
    'AMAZON.NoIntent': function () {
        output = goodbyeMessage;
        alexa.emit(':tell', output);
    },
    'AMAZON.StopIntent': function () {
        this.emit(':tell', goodbyeMessage);
    },
    'AMAZON.RepeatIntent': function () {
        this.emit(':ask', output, HelpMessage);
    },
    'AMAZON.CancelIntent': function () {
        // Use this function to clear up and save any data needed between sessions
        this.emit(":tell", goodbyeMessage);
    },
    'SessionEndedRequest': function () {
        // Use this function to clear up and save any data needed between sessions
    },

    'Unhandled': function () {
        output = HelpMessage;
        this.emit(':ask', output, welcomeRepromt);
    }
});

exports.handler = function (event, context, callback) {
    alexa = Alexa.handler(event, context);
    alexa.registerHandlers(newSessionHandlers, startSearchHandlers, topFiveHandlers);
    alexa.execute();
};

// Create a web request and handle the response.
function httpGet(query, callback) {
  console.log("/n QUERY: "+query);

    var options = {
      //http://api.nytimes.com/svc/search/v2/articlesearch.json?q=seattle&sort=newest&api-key=
        host: 'api.nytimes.com',
        path: '/svc/search/v2/articlesearch.json?q=' + query + '&sort=newest&api-key=' + APIKey,
        method: 'GET'
    };

    var req = http.request(options, (res) => {

        var body = '';

        res.on('data', (d) => {
            body += d;
        });

        res.on('end', function () {
            callback(body);
        });

    });
    req.end();

    req.on('error', (e) => {
        console.error(e);
    });
}

String.prototype.trunc =
      function (n) {
          return this.substr(0, n - 1) + (this.length > n ? '&hellip;' : '');
      };
