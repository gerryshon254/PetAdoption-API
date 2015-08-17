---
---
<!--- Author: Chad Miller -->

<style>
body {
	margin: 3% 10%;
}

label {
	vertical-align: middle;
	margin: auto 1ex;
}

input {
	vertical-align: middle;
}

fieldset {
	padding: 1.2ex;
	float: left;
	border: none;
}

fieldset legend {
	font-size: larger;
}

h1, h2 {
	margin-top: 4ex;
	margin-bottom: 1ex;
	clear: both;
}

ol#petlist {
	padding: 0px;
	margin: 0px;
}

ol#petlist li {
	vertical-align: top;
}

ol#petlist li h3 {
	text-align: center;
}

ol#petlist li {
	display: inline-block;
	width: 20ex;
	height: 35ex;
	border: 2px solid white;
	margin: 0.7ex;
	padding: 0.6ex 1ex;
}

input[type="range"] {
	width: 20ex;
}

</style>

Have a breed of dog in mind? Shelters usually can't attest to the genetics or lineage of a pet, but they can describe the characteristics they observe. If you have a breed in mind, you can [start your search with those breed's features](javascript: display_breed_filler(); false;), to get pets that are closest what you want.


pick traits
-----------

<form id="search" onchange="change(this);" action="javascript: false;">
<fieldset id="shapesfields"><legend>shape</legend></fieldset>
<fieldset id="colorsfields"><legend>colors</legend></fieldset>
<fieldset id="weightlabel"><legend>adult weight</legend></fieldset>
<fieldset id="weightlabel"><legend>known good with*</legend><label><input type="checkbox" id="goodwithkids">&#x1f476;&#x1f6bc;</label><label><input type="checkbox" id="goodwithdogs">&#x1f436;&#x1f415;</label><label><input type="checkbox" id="goodwithcats">&#x1f431;&#x1f408;</label></fieldset>
<fieldset id="energylabel"><legend>energy level</legend><input id="energy" type="range" min="0" max="1" step="0.1"></fieldset>
<fieldset><legend>sex</legend><label><input id="sexf" type="checkbox" checked>female</label> <label><input id="sexm" type="checkbox" checked>male</label></fieldset>
</form>

<p><small>*&nbsp;Stewards&nbsp;at shelters can not judge this very well, so it doesn't have a strong weight in searching.</small></p>

best matches
------------

<ol id="petlist"></ol>

<script>
var max_to_show = 120;
var breed_info;
var breed_aliases;
var pet_info;
var search_is_dirty = true;  // the rendered list doesn't match the search parameters; if in mid-render, stop processing and start over.

var pets = new Array();  // DOM elements in order of pet_info;
var search_result_indexes;  // Indexes into `pets`, ordered by relevance to search results.
var shapes = new Array();
var colors = new Array();

function display_breed_filler() {
	alert("this isn't available yet.");
}

function populatesearch() {
	// TODO  update search from selection
	return false;
}


function change(frm) {
	if (! breed_info) return false;
	if (! pet_info) return false;

	search_is_dirty = true;

	var needle = new Array();
	needle["sexf"] = document.getElementById("sexf").checked;
	needle["sexm"] = document.getElementById("sexm").checked;
	needle["weight"] = document.getElementById("weight").value;
	needle["energy"] = document.getElementById("energy").value;
	for (goodwith in {"kids":1, "dogs":1, "cats":1}) {
		needle["goodwith"+goodwith] = document.getElementById("goodwith"+goodwith).checked;
	}
	var colors_checked = 0;
	for (color in colors) {
		needle["color"+color] = document.getElementById("color"+color).checked;
		colors_checked += needle["color"+color] ? 1 : 0;
	}
	for (shape in shapes) {
		needle["shape"+shape.replace(" ", "")] = document.getElementById("shape"+shape.replace(" ", "")).checked;
	}
	
	scores_for_index = new Array(pet_info.length);
	for (var i = 0; i < pet_info.length; i++) {
		// closeness to searcher's desires. Getting weight right is hard.
		score = 0;
		
		if (needle["sex" + pet_info[i].sex]) { score += 50 }  // sex weights 50
		score += 164 / Math.pow((Math.max(7, Math.abs(needle["weight"] - pet_info[i]["adult_size_lbs"]))/7), 1.5);
		score += 7 / Math.max(0.1, Math.abs(needle["energy"] - pet_info[i]["temperament_active_level"]));
		for (goodwith in {"kids":1, "dogs":1, "cats":1}) {
			if (needle["goodwith"+goodwith] && pet_info[i].known_good_with.indexOf(goodwith)>=0) { score += 20; }
		}
		for (color in colors) {
			if (needle["color" + color]) {
				if (pet_info[i].colors.indexOf(color) >= 0) {
					score += 30;
				}
			}
		}

		for (shape in shape) {
			if (needle["shape" + shape.replace(" ", "")]) {
				if (pet_info[i].shapes.indexOf(shape.replace(" ", "")) >= 0) {
					console.log("found wanted " + shape);
					score += 60;
				}
			}
		}

		score += (i / 100.0);
		scores_for_index[i] = [10000 - score, i];
	}
	
	scores_for_index.sort();

	for (var i = 0; i < max_to_show; i++) {
		search_result_indexes[i] = scores_for_index[i][1];
		console.log(scores_for_index[i]);
	}

	render();
	return false;
}

function render() {
	search_is_dirty = false;

	var ul = document.getElementById("petlist");
	while (ul.firstChild) {
		ul.removeChild(ul.firstChild);
	}

	for (var i = 0; i < max_to_show; i++) {
		if (search_is_dirty) { return false; }
		ul.appendChild(pets[search_result_indexes[i]]);
	}

	return ! search_is_dirty;
}

function post_loading_activate() {
	if (! breed_info) return false;
	if (! pet_info) return false;

	var max_weight;
	var min_weight;

	var pet_display_cells = new Array();
	search_result_indexes = new Array(pet_info.length);

	for (pet in pet_info) {
		if (! max_weight || pet_info[pet]["adult_size_lbs"] > max_weight) { max_weight = pet_info[pet]["adult_size_lbs"]; }
		if (! min_weight || pet_info[pet]["adult_size_lbs"] < min_weight) { min_weight = pet_info[pet]["adult_size_lbs"]; }

		if (pet_info[pet]["shape"]) { shapes[pet_info[pet]["shape"]] = 1; }

		for (color in pet_info[pet]["colors"]) { colors[pet_info[pet]["colors"][color]] = 1; }

		var li = document.createElement("li");
		var name = document.createElement("h3")
		name.appendChild(document.createTextNode(pet_info[pet]["name"]));
		//li.setAttribute("style", "background: url("+pet_info[pet].images_urls[0]+") no-repeat 50% 50%");
		li.appendChild(name);

		li.appendChild(document.createTextNode(pet_info[pet]["colors"].join("/") + " " + pet_info[pet]["shape"]));

		var div = document.createElement("p");
		div.appendChild(document.createTextNode("Activity level: " + (pet_info[pet]["temperament_active_level"]*10) + "/10"));
		li.appendChild(div);

		if (pet_info[pet]["known_good_with"].length > 0) {
			var p = document.createElement("p");
			p.appendChild(document.createTextNode("Known to be good with "));
			for (i in pet_info[pet]["known_good_with"]) {
				p.appendChild(document.createTextNode(pet_info[pet]["known_good_with"][i]));

				if (i < pet_info[pet]["known_good_with"].length-2) {
					p.appendChild(document.createTextNode(", "));
				} else if (i == pet_info[pet]["known_good_with"].length-2) {
					if (pet_info[pet]["known_good_with"].length != 2) {
						p.appendChild(document.createTextNode(","));
					}
					p.appendChild(document.createTextNode(" and "));
				}
			}
			p.appendChild(document.createTextNode("."));
			li.appendChild(p);
		}
		

		pets.push(li);
		search_result_indexes[pet] = pet; // int index

	}

	var weightlabel = document.getElementById("weightlabel");

	var weightslider = document.createElement("input");
	weightslider.setAttribute("id", "weight");
	weightslider.setAttribute("type", "range");
	weightslider.setAttribute("step", "5");
	weightslider.setAttribute("style", "width: auto;");
	weightslider.setAttribute("defaultValue", 0); //Math.round((min_weight+max_weight)/10)*5);
	weightslider.setAttribute("min", min_weight);
	weightslider.setAttribute("max", max_weight);

	var weightsliderindicator = document.createElement("span");
	weightsliderindicator.setAttribute("id", "weightsliderindicator");
	weightsliderindicator.setAttribute("class", "valueindicator");
	weightsliderindicator.appendChild(document.createTextNode("(any)"));

	weightlabel.appendChild(weightsliderindicator);
	weightlabel.appendChild(weightslider);

	weightslider.addEventListener("onchange", (function() { console.log(this); weightsliderindicator.contents = this.value; return true; })());

	var shapesfields = document.getElementById("shapesfields");
	shapes.sort();
	for (shape in shapes) {
		var shapelabel = document.createElement("label");
		var shapecheckbox = document.createElement("input");
		shapecheckbox.setAttribute("id", "shape"+shape.replace(" ", ""));
		shapecheckbox.setAttribute("type", "checkbox");
		shapelabel.appendChild(shapecheckbox);
		shapelabel.appendChild(document.createTextNode(shape));
		shapesfields.appendChild(shapelabel);
	}

	var colorsfields = document.getElementById("colorsfields");
	colors.sort();
	for (color in colors) {
		var colorlabel = document.createElement("label");
		var colorcheckbox = document.createElement("input");
		colorcheckbox.setAttribute("id", "color"+color.replace(" ", ""));
		colorcheckbox.setAttribute("type", "checkbox");
		colorlabel.appendChild(colorcheckbox);
		colorlabel.appendChild(document.createTextNode(color));
		colorsfields.appendChild(colorlabel);
	}

	render();
}

function load_breeds() {
	var b = JSON.parse(breedsxmlhr.responseText);
	breed_info = b["breed_shorthands"];
	breed_aliases = b["breed_aliases"];
	return post_loading_activate();
}

function load_petinfo() {
	var p = JSON.parse(petinfoxmlhr.responseText);
	pet_info = p["animals"];
	return post_loading_activate();
}

var breedsxmlhr = new XMLHttpRequest();
breedsxmlhr.onload = load_breeds;
breedsxmlhr.open("get", "dog_breed_data.json", true);
breedsxmlhr.send();

var petinfoxmlhr = new XMLHttpRequest();
petinfoxmlhr.onload = load_petinfo;
petinfoxmlhr.open("get", "ephemoral_test_data.json", true);
petinfoxmlhr.send();

</script>
