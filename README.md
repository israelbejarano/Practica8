Plataphorma
===========

Meteor pet project created to teach my students the following Meteor functionality: 

* Collection's publish/subscribe 
* Deps.autorun 
* Meteor.methods/call 
* Integration of non-Meteor code in compatibility folder (HTML5 games Alien Invasion and Froot Wars)
* Usage of allow to control client access to collections

![ScreenShot](/screenshot.png)


Plataphorma offers the possibility to run 2 different HTML5 games: Alien Invasion and Froot Wars. 

On the right side of the screen the best players of each game are shown, updated in real time each time a signed in player finishes a game. If no game is selected, the best players overall are shown.

On the left side of the screen a chatroom for the current game is available. Only signed in users can post messages. If no game is selected a general chatroom is shown.

The original code of the two HTML5 games integrated in this project is available here:
* Alien Invasion: https://github.com/cykod/AlienInvasion
* Froot Wars: http://www.wrox.com/WileyCDA/WroxTitle/Professional-HTML5-Mobile-Game-Development.productCd-1118301323,descCd-DOWNLOAD.html

Bootstrap style (file bootstrap.min.css) provided by http://bootswatch.com


Running the project
-------------------

A live version of this code is running here: http://plataphorma.meteor.com

To run the project locally, clone the repo and run ```meteor``` inside it. You can see in .meteor/packages that this Meteor project uses these packages:
* ```meteor remove autopublish```
* ```meteor remove insecure```
* ```meteor add bootstrap```
* ```meteor add accounts-ui```
* ```meteor add accounts-password```


1) Click en botón de juego.

	Template.choose_game.events = {
    	'click #AlienInvasion': function () { // si hacemos click en el boton con id Alieninvasion hace las siguientes lineas
		$('#gamecontainer').hide(); // esconde el div con id gamecontainer que es el div del otro juego
		$('#container').show(); // muestra el div con id container que es donde esta el canvas dle alien invasion
		var game = Games.findOne({name:"AlienInvasion"}); //Con findOne de Meteor encouentra dentro de la coleccion Games el juego con nombre AlienInvasion
		Session.set("current_game", game._id); //Con este comando meteor se inicializa la variable current_game a AlienInvasion.
    	},
    	'click #FrootWars': function () { // si hacemos click en el boton con id FrootWars hace las siguientes lineas
		$('#container').hide(); // esconde el div con id container que es el div del otro juego
		$('#gamecontainer').show(); // muestra el div con id gamecontainer que es donde esta el canvas del froot wars
		var game = Games.findOne({name:"FrootWars"}); //Con findOne de Meteor encouentra dentro de la coleccion Games el juego con nombre Frootwars
		Session.set("current_game", game._id); //Con este comando meteor se inicializa la variable current_game a FrootWars.
    	},
    	'click #none': function () { // si hacemos click en el boton con id none hace las siguientes lineas
		$('#container').hide(); //esconde el div con id container
		$('#gamecontainer').hide(); //esconde el div con id gamecontainer
		Session.set("current_game", "none"); //Con este comando meteor se inicializa la variable current_game a none.
    	}
}

2) Se escribe mensaje chat sin estar autenticado
	
'keydown input#message' : function (event) { // si le doy al intro en input con id message hago lo siguiente
	if (event.which == 13) { //si le doy al intro
	    if (Meteor.userId()){ // como no estoy registrado no tengo Id en meteor por lo tanot voy a la rama else
		var user_id = Meteor.user()._id;	    
		var message = $('#message');
		if (message.value != '') {
		    Messages.insert({
			user_id: user_id,
			message: message.val(),
			time: Date.now(),
			game_id: Session.get("current_game")
		    });
		    message.val('')
		}
	    }
	    else { // muestra el div con id login-error y no nos deja enviar el mensaje
		$("#login-error").show();
	    }
	}
    }
}

3) Se escribe mensaje chat estando autenticado

'keydown input#message' : function (event) { // si le doy al intro en input con id message hago lo siguiente
	if (event.which == 13) { // si le doy al intro
	    if (Meteor.userId()){ // como estoy registrado tengo Id en meteor por lo tanto entro en el if
		var user_id = Meteor.user()._id; // asigno a la variable user_id el valor que hay en meteor con user_id	    
		var message = $('#message'); se asigna a la variable message el valor que hay ne el div con id message
		if (message.value != '') { // si el mensaje no esta vacio
		    Messages.insert({ // inserta en la coleccion messsages los campos pertinentes descritos justo a continuación
			user_id: user_id,
			message: message.val(),
			time: Date.now(),
			game_id: Session.get("current_game")
		    });
		    message.val('')
		}
	    }
	    else {
		$("#login-error").show();
	    }
	}
    }
}

4) Se termina la partida con puntuación alta sin estar autenticado

// Llamada cuando han desaparecido todos los enemigos del nivel sin
// que alcancen a la nave del jugador
var winGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points); // hace la llamada a match finish que esta en server.js
    Game.setBoard(3,new TitleScreen("You win!", 
                                    "Press fire to play again",
                                    playGame));
};


// Llamada cuando la nave del jugador ha sido alcanzada, para
// finalizar el juego
var loseGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points); // hace la llamada a matchfinish que esta en server.js
    Game.setBoard(3,new TitleScreen("You lose!", 
                                    "Press fire to play again",
                                    playGame));
};


Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId) // como indica el comentario arriba si no estoy autenticado no se inserta mi puntuacion. En este caso no lo estoy no se inserta y no s emuestra ne ningun lado dle html.
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
});

5) Se termina la partida con puntuación alta estando autenticado

// Llamada cuando han desaparecido todos los enemigos del nivel sin
// que alcancen a la nave del jugador
var winGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points); // hace la llamada a match finish que esta en server.js
    Game.setBoard(3,new TitleScreen("You win!", 
                                    "Press fire to play again",
                                    playGame));
};


// Llamada cuando la nave del jugador ha sido alcanzada, para
// finalizar el juego
var loseGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points); // hace la llamada a matchfinish que esta en server.js
    Game.setBoard(3,new TitleScreen("You lose!", 
                                    "Press fire to play again",
                                    playGame));
};


Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId) // como indica el comentario arriba si no estoy autenticado no se inserta mi puntuacion. En este caso si lo estoy por lo tanto se guarda en la collecion Matches toda la informacion pertinente.
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
});
