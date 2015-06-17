# Zip-Bot
/**
 * This is the file where the bot commands are located
 *
 * @license MIT license
 */
 
var http = require('http');
var sys = require('sys');
var SetOfficialHunt ='';
 
exports.commands = {
        /**
         * Help commands
         *
         * These commands are here to provide information about the bot.
         */
 
        about: function(arg, by, room, con) {
                if (this.hasRank(by, '#~') || room.charAt(0) === ',') {
                        var text = '';
                } else {
                        var text = '/pm ' + by + ', ';
                }
                text += '**Pokémon Showdown Bot** by: Quinella and TalkTakesTime; **Scavengers commands** by: AnthonySilverstone with help from TheEpicUser';
                this.say(con, room, text);
        },
        help: 'guide',
        guide: function(arg, by, room, con) {
                if (this.hasRank(by, '#~') || room.charAt(0) === ',') {
                        var text = '';
                } else {
                        var text = '/pm ' + by + ', ';
                }
                if (config.botguide) {
                        text += 'A tutorial by Sciz \(Scizornician\) on how to create a PS bot: ' + config.botguide;
                } else {
                        text += 'There is no guide for this bot. PM the owner with any questions.';
                }
                this.say(con, room, text);
        },
 
        /**
         * Dev commands
         *
         * These commands are here for highly ranked users (or the creator) to use
         * to perform arbitrary actions that can't be done through any other commands
         * or to help with upkeep of the bot.
         */
 
        reload: function(arg, by, room, con) {
                if (!this.hasRank(by, '#~')) return false;
                try {
                        this.uncacheTree('./commands.js');
                        Commands = require('./commands.js').commands;
                        this.say(con, room, 'Commands reloaded.');
                } catch (e) {
                        error('failed to reload: ' + sys.inspect(e));
                }
        },
        custom: function(arg, by, room, con) {
                if (!this.hasRank(by, '~')) return false;
                // Custom commands can be executed in an arbitrary room using the syntax
                // ".custom [room] command", e.g., to do !data pikachu in the room lobby,
                // the command would be ".custom [lobby] !data pikachu". However, using
                // "[" and "]" in the custom command to be executed can mess this up, so
                // be careful with them.
                if (arg.indexOf('[') === 0 && arg.indexOf(']') > -1) {
                        var tarRoom = arg.slice(1, arg.indexOf(']'));
                        arg = arg.substr(arg.indexOf(']') + 1).trim();
                }
                this.say(con, tarRoom || room, arg);
        },
        js: function(arg, by, room, con) {
                if (config.excepts.indexOf(toId(by)) === -1) return false;
                try {
                        var result = eval(arg.trim());
                        this.say(con, room, JSON.stringify(result));
                } catch (e) {
                        this.say(con, room, e.name + ": " + e.message);
                }
        },
 
        /**
         * Room Owner commands
         *
         * These commands allow room owners to personalise settings for moderation and command use.
         */
 
        settings: 'set',
        set: function(arg, by, room, con) {
                if (!this.hasRank(by, '%@&#~') || room.charAt(0) === ',') return false;
 
                var settable = {
                        say: 1,
                        joke: 1,
                        choose: 1,
                        usagestats: 1,
                        buzz: 1,
                        helix: 1,
                        survivor: 1,
                        games: 1,
                        wifi: 1,
                        monotype: 1,
                        autoban: 1,
                        happy: 1,
                        guia: 1,
                        studio: 1,
                        'switch': 1,
                        banword: 1,
                        about: 1,
                        scavengerhelp: 1,
                        scavladder: 1,
                        iap: 1,
                        wan: 1,
                        darnell: 1,
                        ashie: 1,
                        spydreigon: 1,
                        anthony: 1,
                        officialhunt: 1,
                        mudp: 1,
                        dsg: 1,
                        zip: 1,
                        snake: 1,
                        glob: 1,
                        pickup: 1,
                        conch: 1,
                        duch: 1,
                        meteordash: 1,
                        gm: 1,
                        gn: 1,
                        tubby: 1
                };
                var modOpts = {
                        flooding: 2,
                        caps: 1,
                        stretching: 1,
                        bannedwords: 2,
                        snen: 2
                };
 
                var opts = arg.split(',');
                var cmd = toId(opts[0]);
                if (cmd === 'mod' || cmd === 'm' || cmd === 'modding') {
                        if (!opts[1] || !toId(opts[1]) || !(toId(opts[1]) in modOpts)) return this.say(con, room, 'Incorrect command: correct syntax is ' + config.commandcharacter + 'set mod, [' +
                                Object.keys(modOpts).join('/') + '](, [on/off])');
 
                        if (!this.settings['modding']) this.settings['modding'] = {};
                        if (!this.settings['modding'][room]) this.settings['modding'][room] = {};
                        if (opts[2] && toId(opts[2])) {
                                if (!this.hasRank(by, '#~')) return false;
                                if (!(toId(opts[2]) in {on: 1, off: 1}))  return this.say(con, room, 'Incorrect command: correct syntax is ' + config.commandcharacter + 'set mod, [' +
                                        Object.keys(modOpts).join('/') + '](, [on/off])');
                                if (toId(opts[2]) === 'off') {
                                        this.settings['modding'][room][toId(opts[1])] = 0;
                                } else {
                                        delete this.settings['modding'][room][toId(opts[1])];
                                }
                                this.writeSettings();
                                this.say(con, room, 'Moderation for ' + toId(opts[1]) + ' in this room is now ' + toId(opts[2]).toUpperCase() + '.');
                                return;
                        } else {
                                this.say(con, room, 'Moderation for ' + toId(opts[1]) + ' in this room is currently ' +
                                        (this.settings['modding'][room][toId(opts[1])] === 0 ? 'OFF' : 'ON') + '.');
                                return;
                        }
                } else {
                        if (!Commands[cmd]) return this.say(con, room, config.commandcharacter + '' + opts[0] + ' is not a valid command.');
                        var failsafe = 0;
                        while (!(cmd in settable)) {
                                if (typeof Commands[cmd] === 'string') {
                                        cmd = Commands[cmd];
                                } else if (typeof Commands[cmd] === 'function') {
                                        if (cmd in settable) {
                                                break;
                                        } else {
                                                this.say(con, room, 'The settings for ' + config.commandcharacter + '' + opts[0] + ' cannot be changed.');
                                                return;
                                        }
                                } else {
                                        this.say(con, room, 'Something went wrong. PM TalkTakesTime here or on Smogon with the command you tried.');
                                        return;
                                }
                                failsafe++;
                                if (failsafe > 5) {
                                        this.say(con, room, 'The command "' + config.commandcharacter + '' + opts[0] + '" could not be found.');
                                        return;
                                }
                        }
 
                        var settingsLevels = {
                                off: false,
                                disable: false,
                                '+': '+',
                                '%': '%',
                                '@': '@',
                                '&': '&',
                                '#': '#',
                                '~': '~',
                                on: true,
                                enable: true
                        };
                        if (!opts[1] || !opts[1].trim()) {
                                var msg = '';
                                if (!this.settings[cmd] || (!this.settings[cmd][room] && this.settings[cmd][room] !== false)) {
                                        msg = '.' + cmd + ' is available for users of rank ' + ((cmd === 'autoban' || cmd === 'banword') ? '#' : config.defaultrank) + ' and above.';
                                } else if (this.settings[cmd][room] in settingsLevels) {
                                        msg = '.' + cmd + ' is available for users of rank ' + this.settings[cmd][room] + ' and above.';
                                } else if (this.settings[cmd][room] === true) {
                                        msg = '.' + cmd + ' is available for all users in this room.';
                                } else if (this.settings[cmd][room] === false) {
                                        msg = '' + config.commandcharacter+''+ cmd + ' is not available for use in this room.';
                                }
                                this.say(con, room, msg);
                                return;
                        } else {
                                if (!this.hasRank(by, '#~')) return false;
                                var newRank = opts[1].trim();
                                if (!(newRank in settingsLevels)) return this.say(con, room, 'Unknown option: "' + newRank + '". Valid settings are: off/disable, +, %, @, &, #, ~, on/enable.');
                                if (!this.settings[cmd]) this.settings[cmd] = {};
                                this.settings[cmd][room] = settingsLevels[newRank];
                                this.writeSettings();
                                this.say(con, room, 'The command ' + config.commandcharacter + '' + cmd + ' is now ' +
                                        (settingsLevels[newRank] === newRank ? ' available for users of rank ' + newRank + ' and above.' :
                                        (this.settings[cmd][room] ? 'available for all users in this room.' : 'unavailable for use in this room.')))
                        }
                }
        },
        blacklist: 'autoban',
        ban: 'autoban',
        ab: 'autoban',
        autoban: function(arg, by, room, con) {
                if (!this.canUse('autoban', room, by) || room.charAt(0) === ',') return false;
                if (!this.hasRank(this.ranks[room] || ' ', '@&#~')) return this.say(con, room, config.nick + ' requires rank of @ or higher to (un)blacklist.');
 
                arg = arg.split(',');
                var added = [];
                var illegalNick = [];
                var alreadyAdded = [];
                if (!arg.length || (arg.length === 1 && !arg[0].trim().length)) return this.say(con, room, 'You must specify at least one user to blacklist.');
                for (var i = 0; i < arg.length; i++) {
                        var tarUser = toId(arg[i]);
                        if (tarUser.length < 1 || tarUser.length > 18) {
                                illegalNick.push(tarUser);
                                continue;
                        }
                        if (!this.blacklistUser(tarUser, room)) {
                                alreadyAdded.push(tarUser);
                                continue;
                        }
                        this.say(con, room, '/roomban ' + tarUser + ', Blacklisted user');
                        this.say(con,room, '/modnote ' + tarUser + ' was added to the blacklist by ' + by + '.');
                        added.push(tarUser);
                }
 
                var text = '';
                if (added.length) {
                        text += 'User(s) "' + added.join('", "') + '" added to blacklist successfully. ';
                        this.writeSettings();
                }
                if (alreadyAdded.length) text += 'User(s) "' + alreadyAdded.join('", "') + '" already present in blacklist. ';
                if (illegalNick.length) text += 'All ' + (text.length ? 'other ' : '') + 'users had illegal nicks and were not blacklisted.';
                this.say(con, room, text);
        },
        unblacklist: 'unautoban',
        unban: 'unautoban',
        unab: 'unautoban',
        unautoban: function(arg, by, room, con) {
                if (!this.canUse('autoban', room, by) || room.charAt(0) === ',') return false;
                if (!this.hasRank(this.ranks[room] || ' ', '@&#~')) return this.say(con, room, config.nick + ' requires rank of @ or higher to (un)blacklist.');
 
                arg = arg.split(',');
                var removed = [];
                var notRemoved = [];
                if (!arg.length || (arg.length === 1 && !arg[0].trim().length)) return this.say(con, room, 'You must specify at least one user to unblacklist.');
                for (var i = 0; i < arg.length; i++) {
                        var tarUser = toId(arg[i]);
                        if (tarUser.length < 1 || tarUser.length > 18) {
                                notRemoved.push(tarUser);
                                continue;
                        }
                        if (!this.unblacklistUser(tarUser, room)) {
                                notRemoved.push(tarUser);
                                continue;
                        }
                        this.say(con, room, '/roomunban ' + tarUser);
                        removed.push(tarUser);
                }
 
                var text = '';
                if (removed.length) {
                        text += 'User(s) "' + removed.join('", "') + '" removed from blacklist successfully. ';
                        this.writeSettings();
                }
                if (notRemoved.length) text += (text.length ? 'No other ' : 'No ') + 'specified users were present in the blacklist.';
                this.say(con, room, text);
        },
        viewbans: 'viewblacklist',
        vab: 'viewblacklist',
        viewautobans: 'viewblacklist',
        viewblacklist: function(arg, by, room, con) {
                if (!this.canUse('autoban', room, by) || room.charAt(0) === ',') return false;
 
                var text = '';
                if (!this.settings.blacklist || !this.settings.blacklist[room]) {
                        text = 'No users are blacklisted in this room.';
                } else {
                        if (arg.length) {
                                var nick = toId(arg);
                                if (nick.length < 1 || nick.length > 18) {
                                        text = 'Invalid nickname: "' + nick + '".';
                                } else {
                                        text = 'User "' + nick + '" is currently ' + (nick in this.settings.blacklist[room] ? '' : 'not ') + 'blacklisted in ' + room + '.';
                                }
                        } else {
                                var nickList = Object.keys(this.settings.blacklist[room]);
                                if (!nickList.length) return this.say(con, room, '/pm ' + by + ', No users are blacklisted in this room.');
                                this.uploadToHastebin(con, room, by, 'The following users are banned in ' + room + ':\n\n' + nickList.join('\n'))
                                return;
                        }
                }
                this.say(con, room, '/pm ' + by + ', ' + text);
        },
        banphrase: 'banword',
        banword: function(arg, by, room, con) {
                if (!this.canUse('banword', room, by)) return false;
                if (!this.settings.bannedphrases) this.settings.bannedphrases = {};
                arg = arg.trim().toLowerCase();
                if (!arg) return false;
                var tarRoom = room;
 
                if (room.charAt(0) === ',') {
                        if (!this.hasRank(by, '~')) return false;
                        tarRoom = 'global';
                }
 
                if (!this.settings.bannedphrases[tarRoom]) this.settings.bannedphrases[tarRoom] = {};
                if (arg in this.settings.bannedphrases[tarRoom]) return this.say(con, room, "Phrase \"" + arg + "\" is already banned.");
                this.settings.bannedphrases[tarRoom][arg] = 1;
                this.writeSettings();
                this.say(con, room, "Phrase \"" + arg + "\" is now banned.");
        },
        unbanphrase: 'unbanword',
        unbanword: function(arg, by, room, con) {
                if (!this.canUse('banword', room, by)) return false;
                arg = arg.trim().toLowerCase();
                if (!arg) return false;
                var tarRoom = room;
 
                if (room.charAt(0) === ',') {
                        if (!this.hasRank(by, '~')) return false;
                        tarRoom = 'global';
                }
 
                if (!this.settings.bannedphrases || !this.settings.bannedphrases[tarRoom] || !(arg in this.settings.bannedphrases[tarRoom]))
                        return this.say(con, room, "Phrase \"" + arg + "\" is not currently banned.");
                delete this.settings.bannedphrases[tarRoom][arg];
                if (!Object.size(this.settings.bannedphrases[tarRoom])) delete this.settings.bannedphrases[tarRoom];
                if (!Object.size(this.settings.bannedphrases)) delete this.settings.bannedphrases;
                this.writeSettings();
                this.say(con, room, "Phrase \"" + arg + "\" is no longer banned.");
        },
        viewbannedphrases: 'viewbannedwords',
        vbw: 'viewbannedwords',
        viewbannedwords: function(arg, by, room, con) {
                if (!this.canUse('banword', room, by)) return false;
                arg = arg.trim().toLowerCase();
                var tarRoom = room;
 
                if (room.charAt(0) === ',') {
                        if (!this.hasRank(by, '~')) return false;
                        tarRoom = 'global';
                }
 
                var text = "";
                if (!this.settings.bannedphrases || !this.settings.bannedphrases[tarRoom]) {
                        text = "No phrases are banned in this room.";
                } else {
                        if (arg.length) {
                                text = "The phrase \"" + arg + "\" is currently " + (arg in this.settings.bannedphrases[tarRoom] ? "" : "not ") + "banned " +
                                        (room.charAt(0) === ',' ? "globally" : "in " + room) + ".";
                        } else {
                                var banList = Object.keys(this.settings.bannedphrases[tarRoom]);
                                if (!banList.length) return this.say(con, room, "No phrases are banned in this room.");
                                this.uploadToHastebin(con, room, by, "The following phrases are banned " + (room.charAt(0) === ',' ? "globally" : "in " + room) + ":\n\n" + banList.join('\n'))
                                return;
                        }
                }
                this.say(con, room, text);
        },
 
        /**
         * General commands
         *
         * Add custom commands here.
         */
 
        tournament: 'tour',
        tour: function(arg, by, room, con) {
                if (room.charAt(0) === ',' || !toId(arg)) return false;
                if (!this.hasRank(this.ranks[room] || ' ', '#~')) {
                        if (!this.hasRank(by, '+%@&#~')) return false;
                        return this.say(con, room, config.nick + " requires # or higher to use the tournament system.");
                }
                arg = arg.split(',');
                if (!this.settings.tourwhitelist) this.settings.tourwhitelist = {};
                if (!this.settings.tourwhitelist[room]) this.settings.tourwhitelist[room] = {};
                if (toId(arg[0]) === 'whitelist') {
                        if (!this.hasRank(by, '&#~')) return false;
                        var action = toId(arg[1] || '');
                        if (!action || action === 'view') {
                                var nickList = Object.keys(this.settings.tourwhitelist[room]);
                                if (!nickList.length) return this.say(con, room, "/pm " + by + ", No users are whitelisted in " + room + ".");
                                return this.uploadToHastebin(con, room, by, "The following users are allowed to control tournaments in " + room + ":\n\n" + nickList.join("\n"));
                        }
                        var target = toId(arg[2] || '');
                        if (!action || !(action in {'add': 1, 'remove': 1}) || !target) return this.say(con, room, "Incorrect syntax: .tour whitelist, [view/add/remove](, [user])");
                        if (action === 'add') {
                                this.settings.tourwhitelist[room][target] = 1;
                                this.say(con, room, "User " + arg[2] + " is now whitelisted and can control tournaments.");
                        } else {
                                if (target in this.settings.tourwhitelist[room]) delete this.settings.tourwhitelist[room][target];
                                this.say(con, room, "User " + arg[2] + " is no longer whitelisted.");
                        }
                        this.writeSettings();
                } else {
                        if (!(this.hasRank(by, (toId(arg[0].split(' ')[0]) in {'dq': 1, 'disqualify': 1} ? '%@' : '') + '&#~') || toId(by) in this.settings.tourwhitelist[room])
                                || toId(arg[0]) in {'join': 1, 'in': 1, 'j': 1}) return false;
                        this.say(con, room, "/tour " + arg.join(','));
                }
        },
        tell: 'say',
        say: function(arg, by, room, con) {
                if (!this.canUse('say', room, by)) return false;
                this.say(con, room, stripCommands(arg) + ' ');
        },
        joke: function(arg, by, room, con) {
                if (!this.canUse('joke', room, by) || room.charAt(0) === ',') return false;
                var self = this;
 
                var reqOpt = {
                        hostname: 'api.icndb.com',
                        path: '/jokes/random',
                        method: 'GET'
                };
                var req = http.request(reqOpt, function(res) {
                        res.on('data', function(chunk) {
                                try {
                                        var data = JSON.parse(chunk);
                                        self.say(con, room, data.value.joke.replace(/&quot;/g, "\""));
                                } catch (e) {
                                        self.say(con, room, 'Sorry, couldn\'t fetch a random joke... :(');
                                }
                        });
                });
                req.end();
        },
        choose: function(arg, by, room, con) {
                if (arg.indexOf(',') === -1) {
                        var choices = arg.split(' ');
                } else {
                        var choices = arg.split(',');
                }
                choices = choices.filter(function(i) {return (toId(i) !== '')});
                if (choices.length < 2) return this.say(con, room, (room.charAt(0) === ',' ? '': '/pm ' + by + ', ') + '.choose: You must give at least 2 valid choices.');
 
                var choice = choices[Math.floor(Math.random()*choices.length)];
                this.say(con, room, ((this.canUse('choose', room, by) || room.charAt(0) === ',') ? '':'/pm ' + by + ', ') + stripCommands(choice));
        },
        usage: 'usagestats',
        usagestats: function(arg, by, room, con) {
                if (this.canUse('usagestats', room, by) || room.charAt(0) === ',') {
                        var text = '';
                } else {
                        var text = '/pm ' + by + ', ';
                }
                text += 'http://sim.smogon.com:8080/Stats/2014-09/';
                this.say(con, room, text);
        },
        seen: function(arg, by, room, con) { // this command is still a bit buggy
                var text = (room.charAt(0) === ',' ? '' : '/pm ' + by + ', ');
                arg = toId(arg);
                if (!arg || arg.length > 18) return this.say(con, room, text + 'Invalid username.');
                if (arg === toId(by)) {
                        text += 'I\'m pretty sure I saw you at Super Weenie Hut Jr.\'s.';
                } else if (arg === toId(config.nick)) {
                        text += 'Who is Cybot?';
                } else if (!this.chatData[arg] || !this.chatData[arg].seenAt) {
                        text += 'The user ' + arg + ' has never been seen.';
                } else {
                        text += arg + ' was last seen ' + this.getTimeAgo(this.chatData[arg].seenAt) + ' ago' + (
                                this.chatData[arg].lastSeen ? ', ' + this.chatData[arg].lastSeen : '.');
                }
                this.say(con, room, text);
 
        },
        pickup: function(arg, by, room, con) {
                if (this.canUse('pickup', room, by) || room.charAt(0) === ',') {
                        var text = '';
                } else {
                        var text = '/pm ' + by + ', ';
                }
 
                var rand = Math.floor(20 * Math.random()) + 1;
 
                switch (rand) {
                        case 1: text += "Are you an interior decorator? Because when I saw you, the entire room became beautiful."; break;
                        case 2: text += "Are you religious? Because you're the answer to all my prayers."; break;
                        case 3: text += "I'm not a photographer, but I can picture me and you together."; break;
                        case 4: text += "Do you have a Band-Aid? Because I just scraped my knee falling for you."; break;
                        case 5: text += "Did you invent the airplane? Cause you seem Wright for me."; break;
                        case 6: text += "If I were a stop light, I'd turn red everytime you passed by, just so I could stare at you a bit longer."; break;
                        case 7: text += "I wanna live in your socks so I can be with you every step of the way."; break;
                        case 8: text += "I thought happiness started with an H. Why does mine start with U?"; break;
                        case 9: text += "Are you a camera? Because every time I look at you, I smile."; break;
                        case 10: text += "Do you have a map? I'm getting lost in your eyes."; break;
                        case 11: text += "I don't have a library card, but do you mind if I check you out?"; break;
                        case 12: text += "Are you a fruit, because Honeydew you know how fine you look right now?"; break;
                        case 13: text += "Does your left eye hurt? Because you've been looking right all day."; break;
                        case 14: text += "Are you a parking ticket? 'Cause you've got fine written all over you."; break;
                        case 15: text += "Was your dad a boxer? Cause you're a knockout!"; break;
                        case 16: text += "If you were a vegetable you'd be a cute-cumber."; break;
                        case 17: text += "If I were a cat I'd spend all 9 lives with you."; break;
                        case 18: text += "Do you work at Starbucks? Because I like you a latte."; break;
                        case 19: text += "Are you a banana? Because I find you a-peeling"; break;
                        case 20: text += "There must be something wrong with my eyes, I can't take them off you."; break;
                }
                this.say(con, room, text);
 
        },
        tubby: function(arg, by, room, con) {
                if (this.canUse('tubby', room, by) || room.charAt(0) === ',') {
                        var text = '';
                } else {
                        var text = '/pm ' + by + ', ';
                        arg = toId(arg);
                if (arg === toId(iap)) {
                        text += 'iap is 100% tubby';
                if (arg === toId(spydreigon)) {
                        text += 'spydreigon is 100% tubby';
 
                }var rand = Math.floor(2 * Math.random()) + 1;
 
                switch (rand) {
                        case 1: text += "arg is 0% tubby"; break;
                        case 2: text += "arg is 100% tubby"; break
 
                }
                this.say(con, room, text);
 
        },
        conch: function(arg, by, room, con) {
                if (this.canUse('conch', room, by) || room.charAt(0) === ',') {
                        var text = '';
                } else {
                        var text = '/pm ' + by + ', ';
                }
 
                var rand = Math.floor(7 * Math.random()) + 1;
 
                switch (rand) {
                        case 1: text += "Maybe someday."; break;
                        case 2: text += "Nothing."; break;
                        case 3: text += "Neither"; break;
                        case 4: text += "I don\'t think so"; break;
                        case 5: text += "Yes."; break;
                        case 6: text += "No."; break;
                        case 7: text += "Try asking again."; break;
                }
                this.say(con, room, text);
        },
 
        /**
         * Room specific commands
         *
         * These commands are used in specific rooms on the Smogon server.
         */
        espaol: 'esp',
        ayuda: 'esp',
        esp: function(arg, by, room, con) {
                // links to relevant sites for the Wi-Fi room
                if (!(room === 'espaol' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('guia', room, by)) {
                        text += '/pm ' + by + ', ';
                }
                var messages = {
                        reglas: 'Recuerda seguir las reglas de nuestra sala en todo momento: http://ps-salaespanol.weebly.com/reglas.html',
                        faq: 'Preguntas frecuentes sobre el funcionamiento del chat: http://ps-salaespanol.weebly.com/faq.html',
                        faqs: 'Preguntas frecuentes sobre el funcionamiento del chat: http://ps-salaespanol.weebly.com/faq.html',
                        foro: '¡Visita nuestro foro para participar en multitud de actividades! http://ps-salaespanol.proboards.com/',
                        guia: 'Desde este índice (http://ps-salaespanol.proboards.com/thread/575/ndice-de-gu) podrás acceder a toda la información importante de la sala. By: Lost Seso',
                        liga: '¿Tienes alguna duda sobre la Liga? ¡Revisa el **índice de la Liga** aquí!: (http://goo.gl/CxH2gi) By: xJoelituh'
                };
                text += (toId(arg) ? (messages[toId(arg)] || '¡Bienvenidos a la comunidad de habla hispana! Si eres nuevo o tienes dudas revisa nuestro índice de guías: http://ps-salaespanol.proboards.com/thread/575/ndice-de-gu') : '¡Bienvenidos a la comunidad de habla hispana! Si eres nuevo o tienes dudas revisa nuestro índice de guías: http://ps-salaespanol.proboards.com/thread/575/ndice-de-gu');
                this.say(con, room, text);
        },
        studio: function(arg, by, room, con) {
                if (!(room === 'thestudio' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('studio', room, by)) {
                        text += '/pm ' + by + ', ';
                }
                var messages = {
                        plug: '/announce The Studio\'s plug.dj can be found here: http://plug.dj/the-studio-3/'
                };
                this.say(con, room, text + (messages[toId(arg)] || ('Welcome to The Studio, a music sharing room on PS!. If you have any questions, feel free to PM a room staff member. Available commands for .studio: ' + Object.keys(messages).join(', '))));
        },
        'switch': function(arg, by, room, con) {
                if (!(room === 'gamecorner' && config.serverid === 'showdown') ||
                        !this.canUse('switch', room, by)) return false;
                this.say(con, room, 'Taking over the world. Starting with Game Corner. Room deregistered.');
                this.say(con, room, '/k ' + (toId(arg) || by) + ', O3O YOU HAVE TOUCHED THE SWITCH');
        },
        wifi: function(arg, by, room, con) {
                // links to relevant sites for the Wi-Fi room
                if (!(room === 'wifi' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('wifi', room, by)) {
                        text += '/pm ' + by + ', ';
                }
                var messages = {
                        intro: 'Here is an introduction to Wi-Fi: http://tinyurl.com/welcome2wifi',
                        rules: 'The rules for the Wi-Fi room can be found here: http://pstradingroom.weebly.com/rules.html',
                        faq: 'Wi-Fi room FAQs: http://pstradingroom.weebly.com/faqs.html',
                        faqs: 'Wi-Fi room FAQs: http://pstradingroom.weebly.com/faqs.html',
                        scammers: 'List of known scammers: http://tinyurl.com/psscammers',
                        cloners: 'List of approved cloners: http://goo.gl/WO8Mf4',
                        tips: 'Scamming prevention tips: http://pstradingroom.weebly.com/scamming-prevention-tips.html',
                        breeders: 'List of breeders: http://tinyurl.com/WiFIBReedingBrigade',
                        signup: 'Breeders Sign Up: http://tinyurl.com/GetBreeding',
                        bans: 'Ban appeals: http://pstradingroom.weebly.com/ban-appeals.html',
                        banappeals: 'Ban appeals: http://pstradingroom.weebly.com/ban-appeals.html',
                        lists: 'Major and minor list compilation: http://tinyurl.com/WifiSheets',
                        trainers: 'List of EV trainers: http://tinyurl.com/WifiEVtrainingCrew',
                        youtube: 'Wi-Fi room\'s official YouTube channel: http://tinyurl.com/wifiyoutube',
                        league: 'Wi-Fi Room Pokemon League: http://tinyurl.com/wifiroomleague'
                };
                text += (toId(arg) ? (messages[toId(arg)] || 'Unknown option. General links can be found here: http://pstradingroom.weebly.com/links.html') : 'Links can be found here: http://pstradingroom.weebly.com/links.html');
                this.say(con, room, text);
        },
        mono: 'monotype',
        monotype: function(arg, by, room, con) {
                // links and info for the monotype room
                if (!(room === 'monotype' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('monotype', room, by)) {
                        text += '/pm ' + by + ', ';
                }
                var messages = {
                        forums: 'The monotype room\'s forums can be found here: http://psmonotypeforum.createaforum.com/index.php',
                        plug: 'The monotype room\'s plug can be found here: http://plug.dj/monotype-3-am-club/',
                        rules: 'The monotype room\'s rules can be found here: http://psmonotype.wix.com/psmono#!rules/cnnz',
                        site: 'The monotype room\'s site can be found here: http://www.psmonotype.wix.com/psmono',
                        league: 'Information on the Monotype League can be found here: http://themonotypeleague.weebly.com/'
                };
                text += (toId(arg) ? (messages[toId(arg)] || 'Unknown option. General information can be found here: http://www.psmonotype.wix.com/psmono') : 'Welcome to the monotype room! Please visit our site to find more information. The site can be found here: http://www.psmonotype.wix.com/psmono');
                this.say(con, room, text);
        },
        survivor: function(arg, by, room, con) {
                // contains links and info for survivor in the Survivor room
                if (!(room === 'survivor' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('survivor', room, by)) {
                        text += '/pm ' + by + ', ';
                }
                var gameTypes = {
                        hg: "http://survivor-ps.weebly.com/hunger-games.html",
                        hungergames: "http://survivor-ps.weebly.com/hunger-games.html",
                        classic: "http://survivor-ps.weebly.com/classic.html"
                };
                arg = toId(arg);
                if (arg) {
                        if (!(arg in gameTypes)) return this.say(con, room, "Invalid game type. The game types can be found here: http://survivor-ps.weebly.com/themes.html");
                        text += "The rules for this game type can be found here: " + gameTypes[arg];
                } else {
                        text += "The list of game types can be found here: http://survivor-ps.weebly.com/themes.html";
                }
                this.say(con, room, text);
        },
        games: function(arg, by, room, con) {
                // lists the games for the games room
                if (!(room === 'gamecorner' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('games', room, by)) {
                        text += '/pm ' + by + ', ';
                }
                this.say(con, room, text + 'Game List: 1. Would You Rather, 2. NickGames, 3. Scattegories, 4. Commonyms, 5. Questionnaires, 6. Funarios, 7. Anagrams, 8. Spot the Reference, 9. Pokemath, 10. Liar\'s Dice');
                this.say(con, room, text + '11. Pun Game, 12. Dice Cup, 13. Who\'s That Pokemon?, 14. Pokemon V Pokemon (BST GAME), 15. Letter Getter, 16. Missing Link, 17. Parameters! More information can be found here: http://psgamecorner.weebly.com/games.html')
 
        },
        scavengerhelp: function(arg, by, room, con) {
         if (!this.canUse('scavengerhelp', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '!scavengerhelp');
        },
        phunt: function(arg, by, room, con) {
         if (!this.canUse('phunt', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '/starthunt 5\+5, 10\, 10\+10\, 20\, 20\+20\, 40');
                this.say(con, room, '/wall Practice Hunt');
        },
        commands: function(arg, by, room, con) {
                //Links to Cybot's command list
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'http://tinyurl.com/cyllagebot All custom commands added to Cybot');
        },
        scavladder: function(arg, by, room, con) {
                //scavenger hunt point ladder
                if (!(room === 'scavengers' && config.serverid === 'showdown')) return false;
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'http://tinyurl.com/scavladder Scavenger Hunt ladder updated courtesy of ashiemore and iap!');
        },
        wan: function(arg, by, room, con) {
                //Wan is Rick
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'My name\'s not Rick! D:');
        },
        rick: function(arg, by, room, con) {
                //Rick is Wan
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'My name\'s not Wan! D:');
        },
        iap: function(arg, by, room, con) {
                //iap is Tubby
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'You\'re goin\' down, Tubby >:(');
        },
        darnell: function(arg, by, room, con) {
                //darnell has a small d
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Emphasis on the small d');
        },
        ashie: function(arg, by, room, con) {
                //a statement about ashiemore
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'ashie is a scrub');
        },
        spydreigon: function(arg, by, room, con) {
                //sixtynein
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + '69');
        },
        anthony: function(arg, by, room, con) {
                //creator
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Anthony Silverstone is my divine creator. All hail Anthony Silverstone! o/');
        },
        kevinrocks: function(arg, by, room, con) {
                //nub
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Some nub');
        },
        mudp: function(arg, by, room, con) {
         if (!this.canUse('mudp', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '!learn muk\, mimic ');
        },
        dsg: function(arg, by, room, con) {
                //declaration that iap is actually Tubby
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'When it comes to old trivia, DSG is [[Keeping The Faith-Billy Joel]]');
        },
        zip: function(arg, by, room, con) {
                //declaration that iap is actually Tubby
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Zipzapadam: Quality Hunts since 1969');
        },
        kevin: function(arg, by, room, con) {
         if (!this.canUse('snake', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '[[Hi Kevin]]');
        },
        snake: function(arg, by, room, con) {
         if (!this.canUse('snake', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '/me trips over snakeindagrass');
        },
        glob: function(arg, by, room, con) {
                //declaration that Gleeb is a Trivia nerd
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Gleeb is a Trivia nerd! \>\:D');
        },
        dylas: function(arg, by, room, con) {
                //makes Cybot ask when the official hunt will be
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'When is official?');
        },
        duch: function(arg, by, room, con) {
                //links to what is currently the longest port in scavengers history
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'http://pastebin.com/9qqMQH3U');
        },
        meteordash: function(arg, by, room, con) {
                //rip
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'rip ;-;7');
        },     
        gm: function(arg, by, room, con) {
                //Good morning
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Good morning, Scavengers!');
        },     
        gn: function(arg, by, room, con) {
                //Good night
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Good night\, Scavengers!');
        },     
        officialhunt: function(arg, by, room, con) {
                //Good night
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'The next Official Hunt will be at 7:00 PM GMT by %losedude/');
        },
        lh: function(arg, by, room, con) {
                //LiepardLover
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + '\[06\:09\:24\] \+LiepardHater\: My name is a lie. Liepard is my favourite pokemon.');
        },
        snaq: function(arg, by, room, con) {
                //Victini Q
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Which Pokemon were seen while you could see Victini in Pokemon the Movie White - White Victini and Zekrom');
        },
        leaking: function(arg, by, room, con) {
                if (!this.canUse('leaking', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '**Giving/leaking answers during the official hunt will result in a room ban. Use the appropriate commands while participating in the hunt.**');
        },
        ded: function(arg, by, room, con) {
                if (!this.canUse('ded', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '!dt dedchat');
        },
        shituser: function(arg, by, room, con) {
                //shituser
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'Shit User is your lord and saviour');
        },
        scavfaq: function(arg, by, room, con) {
                //Scavengers faq
                var text = '';
                if (!this.canUse('scavengers', room, by)) text+= "/pm " + by + ", ";
                this.say(con, room, text + 'http://pastebin.com/F3KMK5bB Scavengers FAQ');
        },
        scrub: function(arg, by, room, con) {
                if (!this.canUse('scrub', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '\/w ashiemore\, Scrub.'); 
        },
        cido: function(arg, by, room, con) {
                if (!this.canUse('cido', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '\/w cido\, Congrats on global voice!');
        },     
        angels: function(arg, by, room, con) {
                if (!this.canUse('angels', room, by) || room.charAt(0) === ',') return false;
                this.say(con, room, '\/me gives angels2002 a Big Mac.');
        },
};
