<html>
    <head>
        <meta charset="utf-8">
        <title>ML Accuracy Calculator</title>
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script> 
        <style>
            table, th, td{
                border: 2px solid black;
                margin-left:auto;
                margin-right:auto;
                text-align: center;
            }
            th, td,img{
                max-height: 200px;
                max-width: 200px;
            }
            input,select{
                width:100%;
            }
            input{
                text-align: right;
            }
            img { 
	            display: block;
	            margin-left: auto;
	            margin-right: auto;
            }
        </style>
        <script>
            $(document).ready(function(){
                jQuery.expr[':'].contains = function(a, i, m) {
		            return jQuery(a).text().toUpperCase().indexOf(m[3].toUpperCase()) >= 0;
	            };
                $(document).on('focus','#search',function() {
                    var inp = $(this).val();
                    if (inp == 'Search...')
                        $(this).val('');
                });
                $(document).on('focusout','#search',function() {
                    var inp = $(this).val();
                    if (inp == '')
                    {
                        $(this).val('Search...');    
                    }
                });
                $(document).on('keyup','#search',function(){
                    var inp = $(this).val().toLowerCase();
                    if(inp != '') {
                        $('#mobs').find(':hidden').show();
                        $('#mobs').find(':not(:contains(' + inp + '))').hide();
                    }
                });
            });
            function clearForm(){
                document.calc.reset();
                document.getElementById("luk").disabled=true;
                document.getElementById("dmgType").innerHTML='ACC:';
                document.getElementById("liblink").href='';
                document.getElementById("mobPic").src='';
                document.getElementById('HP').innerHTML = '-';
                document.getElementById('EXP').innerHTML = '-';
                document.getElementById('weak').innerHTML = '-';
                document.getElementById('strong').innerHTML = '-';
                document.getElementById('immune').innerHTML = '-';
                $('#mobs').empty().append('<option value="' + "null" + '">' + 'Select a world' + '</option>');
                
            }
            var loadedjson;
            function elementChecker(array){
                let arr = new Array();
                if(array[0])
                    arr.push('Ice');
                if(array[1])
                    arr.push('Lightning');
                if(array[2])
                    arr.push('Fire');
                if(array[3])
                    arr.push('Poison')
                if(array[4])
                    arr.push('Holy');
                if(arr.length == 0)
                    arr.push('-');
                return arr;
            }
            function dmgType(type) {
				if(type == 'INT:')
					document.getElementById("luk").disabled=false;
				else
					document.getElementById("luk").disabled=true;
				document.getElementById("dmgType").innerHTML=type;
            }
            function worldSelect(world){
				$.getJSON('https://mrsoupman.github.io/Maple-ACC-calculator/Monsters/' + world + '.json',function(data){
					loadedjson = data;
					$('#mobs').empty();
					$.each(data, function(key, value) {
						$('#mobs').append('<option value="' + key + '">' + "(" + value["level"] + ") " + key + '</option>');
					});
				});	
            }
            function mobSelect(mob){
				if(mob != "null"){
                    document.getElementById('mobPic').src = 'https://mrsoupman.github.io/Maple-ACC-calculator/images/' + loadedjson[mob]["id"] + '.png';
                    document.getElementById('mobLevel').value = loadedjson[mob]["level"];
                    document.getElementById('mobAvoid').value = loadedjson[mob]["avoid"];
                    document.getElementById('HP').innerHTML = loadedjson[mob]["hp"];
                    document.getElementById('EXP').innerHTML = loadedjson[mob]["exp"];
                    document.getElementById('liblink').href = 'https://maplelegends.com/lib/monster?id=' + loadedjson[mob]["id"];
                    document.getElementById('weak').innerHTML = elementChecker(loadedjson[mob]['weak']).toString();
                    document.getElementById('strong').innerHTML = elementChecker(loadedjson[mob]['strong']).toString();
                    document.getElementById('immune').innerHTML = elementChecker(loadedjson[mob]['immune']).toString();
				}
            }
            function sortByLevel(){
                var options = $('#mobs option');
                options.sort(function(a,b) {
                    if(loadedjson[a.value]["level"] > loadedjson[b.value]["level"]) return 1;
                    if(loadedjson[a.value]["level"] < loadedjson[b.value]["level"]) return -1;
                    return 0;
                })
                $('#mobs').empty().append(options);
            }

            function sortByAlpha(){
                var options = $('#mobs option');
                options.sort(function(a,b) {

                    if(a.value > b.value) return 1;
                    if(a.value < b.value) return -1;
                    return 0;
                })
                $('#mobs').empty().append(options);
            }

            function doSomeMath(){
                var monLevel = parseInt(document.getElementById('mobLevel').value);
                var monAvoid = parseInt(document.getElementById('mobAvoid').value);
                var charLevel = parseInt(document.getElementById('level').value);
                var charMainStat = parseInt(document.getElementById('mainstat').value);
                var charLuk = parseInt(document.getElementById('luk').value);
                var diff = monLevel - charLevel;
                var acc100 = 0;
                var acc1 = 0;
                var AccRatio = 0;
                if(diff < 0)
                    diff = 0;
                if(document.querySelector('input[name="type"]:checked').id == 'physical') {
                    acc100 = (55.2 + 2.15 * diff) * (monAvoid/15.0)
                    acc1 = acc100 * 0.5 + 1;
                    AccRatio = 100 * ((charMainStat - (acc100 * 0.5) ) / (acc100 * 0.5));
                }
                else { 
                    var curAcc=( Math.floor(charMainStat/10) + Math.floor(charLuk/10) );
                    acc100 = Math.floor((monAvoid + 1.0)*(1.0 + (0.04 * diff)));
                    acc1 = Math.round(0.41*acc100);
                    AccPart = (curAcc-acc1+1)/(acc100-acc1+1);
                    AccRatio = ((-0.7011618132 * Math.pow(AccPart,2)) + (1.702139835 * AccPart)) * 100;
                }
                if(AccRatio > 100)
                    AccRatio = 100;
                 else if(AccRatio < 0)
                    AccRatio = 0;
                document.getElementById('mob1acc').value = acc1.toPrecision(3);
                document.getElementById('mob100acc').value = acc100.toPrecision(3);
                document.getElementById('mobRate').value = AccRatio.toPrecision(3) + "%";
            }
        </script>
    </head>
    <body onload="clearForm()">
        <form name="calc">
            <table style="margin-left:auto; margin-right: auto;">
                <tr>
                    <th>Monster's stats</th>
                    <th>Character Stats</th>
                </tr>
                <tr>
                    <td>
                        <label for="mobLevel">Level:</label>
                        <input type="number" id="mobLevel" min="1" max ="200" value="1">
                        <br>
                        <label for="mobAvoid">Avoid:</label>
                        <input type="number" id="mobAvoid" min="0" max ="999" value="0">
                        
                    </td>
                    <td rowspan="2">
                        <label for="level">Level:</label>
					    <input type="number" id="level" min="1" max ="200" value="1">
					    <br>
					    <input type="radio" id="physical" name="type" onclick="dmgType('ACC:')" checked style="width:auto">
					    <label for="physical">Physical</label>
					    <br>
					    <input type="radio" id="magical" name="type" onclick="dmgType('INT:')" style="width:auto">
					    <label for="magical">Magical</label>
					    <br>
					    <label for="mainstat" id="dmgType">ACC:</label>
					    <input type="number" id="mainstat" min="4" max ="999" value="4">
					    <br>
					    <label for="luk" id="dmgType">LUK:</label>
					    <input type="number" id="luk" disabled="true" min="4" max ="999" value="4">
                    </td>
                </tr>
                <tr>
                    <td>
                        <select size="7" class="areas" onchange="worldSelect(this.value)">
                            <option value="all">All Worlds</option>
                            <option value="Bosses">Bosses</option>
                            <option value="Aqua">Aqua Road</option>
                            <option value="China">China</option>
                            <option value="LudusLake">Ludibrium/KFT/Omega</option>
                            <option value="Masteria">Masteria</option>
                            <option value="Minar">Minar</option>
                            <option value="MuLung">Mu Lung</option>
                            <option value="Neotokyo">Neotokyo</option>
                            <option value="Nihal">Ariant/Magatia</option>
                            <option value="OrbNath">Orbis/El Nath</option>
                            <option value="PQ">PQ/Job</option>
                            <option value="Singapore">Singapore</option>
                            <option value="Taiwan">Taiwan</option>
                            <option value="Thailand">Thailand</option>
                            <option value="ToT">Temple of Time</option>
                            <option value="VicIsland">Victoria Island</option>
                            <option value="Zipangu">Zipangu</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <td>
                        <button type="button" onclick="sortByLevel()">Level</button>
                        <button type="button" onclick="sortByAlpha()">Alpha</button>
                        <br>
                        <input autocomplete="off" id="search" size="9" value="Search..." style="text-align: left;"></input>
                        <br>
                        <select class="mobs" size="7" name="mobs" id="mobs" onchange="mobSelect(this.value)">
                            <option value="null">Select a world</option>
                        </select>
                    </td>
                    <td>
                        <label for="mob1acc">Accuracy for 1%:</label>
                        <input type="text" id="mob1acc" size="10" disabled="true">
                        <br>
                        <label for="mob100acc">Accuracy for 100%:</label>
                        <input type="text" id="mob100acc" size="8" disabled="true">
                        <br>
                        <label for="mobRate">Hit rate:</label>
                        <input type="text" id="mobRate" size="17" disabled="true">
                    </td>
                </tr>
                <tr style="height: 200px; width: 200px;">
                    <td>
                        <a id="liblink" target="_blank">
                            <img id="mobPic" src=""/>
                        </a>   
                    </td>
                    <td>
                        <label for="weak">Weak:</label>
                        <label id="weak">-</label>
                        <br>
                        <label for="strong">Strong:</label>
                        <label id="strong">-</label>
                        <br>
                        <label for="immune">Immune:</label>
                        <label id="immune">-</label>
                        <br>
                        <label for="HP">HP:</label>
                        <label id="HP">-</label>
                        <br>
                        <label for="EXP">EXP:</label>
                        <label id="EXP">-</label>
                        <br>
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <button type="button" onclick="clearForm()">Reset</button>
                        <button type="button" onclick="doSomeMath()">Calculate</button>
                    </td>
                </tr>
            </table>
        </form>
        <p style="text-align: center; font-size: smaller;">Special thanks to Screaming Statue and Nekonecat for their accuracy calculators, ayumilove and 安心 for their formulas, Nani for being the best guild and Treehouse for being the best alliance <3</p>
    </body>
</html>
