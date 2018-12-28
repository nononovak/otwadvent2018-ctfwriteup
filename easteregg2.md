# easteregg2 - web (50)

I only got this one after the fact with a large hint from hpmv2. It was found base64 encoded in one of the included scripts

```
$ curl https://advent2018.overthewire.org/static/js/app.js

...

  var checkNewChallenges = function() {
	var lockedcount = 0;
	for (var i = 0; i < $scope.challenges.length; i++) {
	  if($scope.challenges[i]["state"] == "locked") {
		lockedcount += 1;
	  }
	}

	console.log("scope lockedcount = "+$scope.lockedCount+" and lockedcount = "+lockedcount);

	if(lockedcount != $scope.lockedCount) {
	  if($scope.lockedCount != -1) {
	        if(localStorage.getItem("gfx") != 2) {
		    var audio = new Audio('/static/audio/santabells.mp3');
		    audio.play();
		    $scope.showsanta = true;
			$("#santa").html(atob("PHA+QU9UV3tKaW5nbGVfQWxsX1RoZV9XYXkhISExfTwvcD4="));
		    console.log("GFX enabled");
		} else {
		    console.log("GFX disabled");
		}
		console.log("Detected new unlocked challenges");
	  }
	  $scope.lockedCount = lockedcount;
    }
  }

...
```

The base64 blob in the middle there decodes as the flag:

```
$ echo 'PHA+QU9UV3tKaW5nbGVfQWxsX1RoZV9XYXkhISExfTwvcD4=' | base64 -D
<p>AOTW{Jingle_All_The_Way!!!1}</p>
```

