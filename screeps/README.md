# Screeps
[Official Website](https://screeps.com)

What is Screeps?

Screeps is a massive multiplayer online real-time strategy game. Each player can create his or her own colony in a single persistent world shared by all the players. Such a colony can mine resources, build units, conquer territory. As you conquer more territory, your influence in the game world grows, as well as your abilities to expand your footprint. However, it requires a lot of effort on your part, since multiple players may aim at the same territory.[[1]](http://docs.screeps.com/introduction.html#What-kind-of-game-is-Screeps)

## Setup environment
* IDE Used: WebStorm

## Local host or server
Firstly, I would setup on localhost to learn. Then, once I done the tutorial then I go to screeps server.

## Start choosing a base
Normally we need to consider 2 things, which is the nearest enemy and the amount of resources

## Start with harvester
Spawn name: Base

### Spawn Creep from base with the use of in game console
(PS: This creep has double the speed with MOVE, MOVE)

`Game.spawns['Base'].createCreep([WORK, CARRY, MOVE, MOVE], "Harvester1");`

__Body Part of the creep__
* WORK – ability to harvest energy, construct and repair structures, upgrade controllers.
* MOVE – ability to move.
* CARRY – ability to transfer energy.
* ATTACK – ability of short-range attack.
* RANGED_ATTACK – ability of ranged attack.
* HEAL – ability to heal others.
* CLAIM - ability to claim territory control.
* TOUGH – "empty" part with the sole purpose of defense.
[[2]](http://docs.screeps.com/creeps.html)

__Global Object__

__[`Game` Object](http://docs.screeps.com/global-objects.html#Game-object)__

__[`Memory` Object](http://docs.screeps.com/global-objects.html#Memory-object)__

### Make creep collect resource
in [`01.js`](./01.js)


	module.exports.loop = function () {
		var creep = Game.creeps['Harvester1'];
		var sources = creep.room.find(FIND_SOURCES);
		if(creep.harvest(sources[0] == ERR_NOT_IN_RANGE)) {
			creep.moveTo(sources[0]);
		}
	}

### Make creep transfer resource back to Spawn
in [`02.js`](./02.js)

	module.exports.loop = function () {
	    var creep = Game.creeps['Harvester1'];

	    if(creep.carry.energy < creep.carryCapacity) {
	        var sources = creep.room.find(FIND_SOURCES);
	        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	            creep.moveTo(sources[0]);
	        }
	    }
	    else {
	        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
	            creep.moveTo(Game.spawns['Spawn1']);
	        }
	    }
	}

### Control Multiple creep
in [03.js](./03.js)

	module.exports.loop = function () {
	    for(var name in Game.creeps) {
	        var creep = Game.creeps[name];

	        if(creep.carry.energy < creep.carryCapacity) {
	            var sources = creep.room.find(FIND_SOURCES);
	            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(sources[0]);
	            }
	        }
	        else {
	            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(Game.spawns['Spawn1']);
	            }
	        }
	    }
	}

### Create Harvester
using module (in harvester module)

[04.js in roleHarvester](./04.js)

	var roleHarvester = {

	    /** @param {Creep} creep **/
	    run: function(creep) {
		    if(creep.carry.energy < creep.carryCapacity) {
	            var sources = creep.room.find(FIND_SOURCES);
	            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(sources[0]);
	            }
	        }
	        else {
	            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(Game.spawns['Spawn1']);
	            }
	        }
		}
	};

	module.exports = roleHarvester;

### in main.js after creating role harvester module
[05.js](./05.js)

	var roleHarvester = require('role.harvester');

	module.exports.loop = function () {

	    for(var name in Game.creeps) {
	        var creep = Game.creeps[name];
	        roleHarvester.run(creep);
	    }
	}

## Upgrading Controller with upgrader
in console, start create an upgrader

`Game.spawns['Spawn1'].createCreep( [WORK, CARRY, MOVE], 'Upgrader1' );`

Give them role name with `MEMORY`

`Game.creeps['Harvester1'].memory.role = 'harvester';
Game.creeps['Upgrader1'].memory.role = 'upgrader';`

### Upgrader role
in [06.js](./06.js)

	var roleUpgrader = {

	    /** @param {Creep} creep **/
	    run: function(creep) {
		    if(creep.carry.energy == 0) {
	            var sources = creep.room.find(FIND_SOURCES);
	            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(sources[0]);
	            }
	        }
	        else {
	            if(creep.upgradeController(creep.room.controller) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(creep.room.controller);
	            }
	        }
		}
	};

	module.exports = roleUpgrader;

### include roleUpgrader in main file
in [07.js](./07.js)
	var roleHarvester = require('role.harvester');
	var roleUpgrader = require('role.upgrader');

	module.exports.loop = function () {

	    for(var name in Game.creeps) {
	        var creep = Game.creeps[name];
	        if(creep.memory.role == 'harvester') {
	            roleHarvester.run(creep);
	        }
	        if(creep.memory.role == 'upgrader') {
	            roleUpgrader.run(creep);
	        }
	    }
	}

## Building Structure
Create a builder:

`Game.spawns['Spawn1'].createCreep( [WORK, CARRY, MOVE], 'Builder1',
    { role: 'builder' } );`

### Create a builder module
in [08.js](./08.js)

	var roleBuilder = {

	    /** @param {Creep} creep **/
	    run: function(creep) {

		    if(creep.memory.building && creep.carry.energy == 0) {
	            creep.memory.building = false;
	            creep.say('🔄 harvest');
		    }
		    if(!creep.memory.building && creep.carry.energy == creep.carryCapacity) {
		        creep.memory.building = true;
		        creep.say('🚧 build');
		    }

		    if(creep.memory.building) {
		        var targets = creep.room.find(FIND_CONSTRUCTION_SITES);
	            if(targets.length) {
	                if(creep.build(targets[0]) == ERR_NOT_IN_RANGE) {
	                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
	                }
	            }
		    }
		    else {
		        var sources = creep.room.find(FIND_SOURCES);
	            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
	            }
		    }
		}
	};

	module.exports = roleBuilder;

in main file [09.js](./09.js)

	var roleHarvester = require('role.harvester');
	var roleBuilder = require('role.builder');

	module.exports.loop = function () {

	    for(var name in Game.creeps) {
	        var creep = Game.creeps[name];
	        if(creep.memory.role == 'harvester') {
	            roleHarvester.run(creep);
	        }
	        if(creep.memory.role == 'builder') {
	            roleBuilder.run(creep);
	        }
	    }
	}

### Multi function harvester
[10.js](./10.js)

	var roleHarvester = {

	    /** @param {Creep} creep **/
	    run: function(creep) {
		    if(creep.carry.energy < creep.carryCapacity) {
	            var sources = creep.room.find(FIND_SOURCES);
	            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
	            }
	        }
	        else {
	            var targets = creep.room.find(FIND_STRUCTURES, {
	                    filter: (structure) => {
	                        return (structure.structureType == STRUCTURE_EXTENSION || structure.structureType == STRUCTURE_SPAWN) &&
	                            structure.energy < structure.energyCapacity;
	                    }
	            });
	            if(targets.length > 0) {
	                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
	                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
	                }
	            }
	        }
		}
	};

	module.exports = roleHarvester;


Since there is a lot of resources, build someone "bulky"

`Game.spawns['Spawn1'].createCreep( [WORK,WORK,WORK,WORK,CARRY,MOVE,MOVE],
    'HarvesterBig',
    { role: 'harvester' } );`

## Automate Task
Find out the number of certain creep

	var roleHarvester = require('role.harvester');
	var roleUpgrader = require('role.upgrader');

	module.exports.loop = function () {

	    var harvesters = _.filter(Game.creeps, (creep) => creep.memory.role == 'harvester');
	    console.log('Harvesters: ' + harvesters.length);

	    for(var name in Game.creeps) {
	        var creep = Game.creeps[name];
	        if(creep.memory.role == 'harvester') {
	            roleHarvester.run(creep);
	        }
	        if(creep.memory.role == 'upgrader') {
	            roleUpgrader.run(creep);
	        }
	    }
	}
	

### if harvester < 2
[12.js](./12.js)

	var roleHarvester = require('role.harvester');
	var roleUpgrader = require('role.upgrader');

	module.exports.loop = function () {

	    var harvesters = _.filter(Game.creeps, (creep) => creep.memory.role == 'harvester');
	    console.log('Harvesters: ' + harvesters.length);

	    if(harvesters.length < 2) {
	        var newName = Game.spawns['Spawn1'].createCreep([WORK,CARRY,MOVE], undefined, {role: 'harvester'});
	        console.log('Spawning new harvester: ' + newName);
	    }
	    
	    if(Game.spawns['Spawn1'].spawning) { 
	        var spawningCreep = Game.creeps[Game.spawns['Spawn1'].spawning.name];
	        Game.spawns['Spawn1'].room.visual.text(
	            '🛠️' + spawningCreep.memory.role,
	            Game.spawns['Spawn1'].pos.x + 1, 
	            Game.spawns['Spawn1'].pos.y, 
	            {align: 'left', opacity: 0.8});
	    }

	    for(var name in Game.creeps) {
	        var creep = Game.creeps[name];
	        if(creep.memory.role == 'harvester') {
	            roleHarvester.run(creep);
	        }
	        if(creep.memory.role == 'upgrader') {
	            roleUpgrader.run(creep);
	        }
	    }
	}

### To test the code, suicide your creep 😆
`Game.creeps['Harvester1'].suicide()`

### Since that suicided creep is still in memory, we need to clear it 😵
	for(var name in Memory.creeps) {
	    if(!Game.creeps[name]) {
	        delete Memory.creeps[name];
	        console.log('Clearing non-existing creep memory:', name);
	    }
	}

### `renewCreep`
Increase the remaining time to live of the target creep.

[Docs](http://docs.screeps.com/api/#StructureSpawn.renewCreep)

## Defend of the Screeps
### Safe Mode
In safe mode, no other creep will be able to use any harmful methods in the room (but you’ll still be able to defend against strangers).

`Game.spawns['Spawn1'].room.controller.activateSafeMode();`

### Build Tower
__`createConstructionSite(x, y, structureType)
(pos, structureType)`__


In console,

`Game.spawns['Spawn1'].room.createConstructionSite( 23, 22, STRUCTURE_TOWER );`



### Harvester sent resource to tower
	var roleHarvester = {

	    /** @param {Creep} creep **/
	    run: function(creep) {
		    if(creep.carry.energy < creep.carryCapacity) {
	            var sources = creep.room.find(FIND_SOURCES);
	            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
	                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
	            }
	        }
	        else {
	            var targets = creep.room.find(FIND_STRUCTURES, {
	                    filter: (structure) => {
	                        return (structure.structureType == STRUCTURE_EXTENSION ||
	                                structure.structureType == STRUCTURE_SPAWN ||
	                                structure.structureType == STRUCTURE_TOWER) && structure.energy < structure.energyCapacity;
	                    }
	            });
	            if(targets.length > 0) {
	                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
	                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
	                }
	            }
	        }
		}
	};

	module.exports = roleHarvester;

### Tower ready
Tower can `Heal`, `Attack` and `Repair`

ATTACK!

	var tower = Game.getObjectById('8250e3458103bc0e4a356a19');
	    if(tower) {
	        var closestHostile = tower.pos.findClosestByRange(FIND_HOSTILE_CREEPS);
	        if(closestHostile) {
	            tower.attack(closestHostile);
	        }
	    }

+HEAL!!!!

	var tower = Game.getObjectById('8250e3458103bc0e4a356a19');
	    if(tower) {
	        var closestDamagedStructure = tower.pos.findClosestByRange(FIND_STRUCTURES, {
	            filter: (structure) => structure.hits < structure.hitsMax
	        });
	        if(closestDamagedStructure) {
	            tower.repair(closestDamagedStructure);
	        }

	        var closestHostile = tower.pos.findClosestByRange(FIND_HOSTILE_CREEPS);
	        if(closestHostile) {
	            tower.attack(closestHostile);
	        }
	    }

## Resources / Source
[1] http://docs.screeps.com/introduction.html#What-kind-of-game-is-Screeps

[2] http://docs.screeps.com/creeps.html


