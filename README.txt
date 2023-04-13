general approach:
    1) Explore posibilities to interact with dota api
    2) Opendota seems to be the only reliable version
        - limitations in form of requests per minute etc
        - decent documentation, but some variables very unclear
        - getting matches by league id returns a reduced dataset
        - using league id to get relevant match id and then use dedicated match methods
    3) Load a match and explore the data a little bit
        - simple graphs, stats, test etc.
    4) Download "Lima Major" matches
        - keep everything and dump raw data to a json-file
    5) Build data cleaning pipeline
        - read data from json and create pandas dataframe -> 197x48 in size
        - keep non-list features as they are for now
        - list features need special care:
            * 'chat' only interesting for chatwheel usage -> opendota does not have information 
              for new chat wheel sounds -> empty on website
              https://github.com/odota/dotaconstants/blob/master/build/chat_wheel.json
              link to json with all chatwheel options -> sprays + audio
            * 'cosmetics' not interested in shiny pixels
              DROP
            * 'draft_timings' not interesting -> 'picks_bans' sufficient
              DROP
            * 'objectives' may contain some additional information
              contains: 'building_kill', 'AEGIS', 'AEGIS_STOLEN', 'COURIER_LOST', 'FIRST_BLOOD',
                        'ROSHAN_KILL'
              all useable
            * 'picks_bans' very useful for pick priority, team analysis
            * 'radiant_gold_adv', 'radiant_exp_adv' very useful for graphs -> mean, std etc.
            * 'team_fights' doesn't seem to be a reliable stat, maybe do something in the end
              DROP
            * 'league' is useless if all matches have the same
              DROP
            * 'radiant_team' & 'dire_team' useful for visualization -> name, image, id
            * 'players' huge entry -> need own dataframe with advanced stats
            * 'all_word_counts' & 'my_word_counts' useless
              DROP
        - inspect non-list features
            * 'match_id' keep in case splitting results in mulitple tables
            * 'barracks_status_radiant/dire' endgame status of buildings -> objectives
              more interesting
              DROP
            * 'dire_score' & 'radiant_score' useful
            * 'dire_team_id' & 'radiant_team_id' already covered by 'radiant_team' & 'dire_team'
              DROP
            * 'cluster' all same value -> useless
              DROP
            * 'duration' useful
            * 'engine' useless
              DROP
            * 'first_blood_time' covered in 'objectives'
              DROP
            * 'game_mode' all the same
              DROP
            * 'human_players' number of human players -> useless
              DROP
            * 'leagueid' all the same
              DROP
            * 'lobby_type' all the same
              DROP
            * 'match_seq_num' useless
              DROP
            * 'negative_votes' & 'positive_votes' dont care
              DROP
            * 'radiant_win' very good
            * 'skill' none
              DROP
            * 'start_time' dont care
              DROP
            * 'tower_status_radiant' & 'tower_status_dire' useless
              DROP
            * 'version' dont care
              DROP
            * 'replay_salt' dont care
              DROP
            * 'series_id' & 'series_type' dont care
              DROP
            * 'patch' & 'region' dont care
              DROP
            * 'throw' & 'loss' dont care
              DROP
            * 'replay_url' dont care
              DROP
            * 'comeback' & 'stomp' dont care
              DROP
            * it probably would have been easier to define the features to keep ...
        - start playing around with list features and gather ideas about processing
        - ...
                        
                        

EXPLORATION - list objects, quick overview + ideas
features:
    1) objectives:
        - 'CHAT_MESSAGE_AEGIS'        -> keys: time, slot, player_slot
        - 'CHAT_MESSAGE_AEGIS_STOLEN' -> keys: time, slot, player_slot
        - 'CHAT_MESSAGE_COURIER_LOST' -> keys: time, value, killer, team
        - 'CHAT_MESSAGE_FIRSTBLOOD'   -> keys: time, slot, key, player_slot
        - 'CHAT_MESSAGE_ROSHAN_KILL'  -> keys: time, team
        - 'building_kill'             -> keys: time, unit, key, slot, player_slot
        - keys:
            * time: seconds
            * slot: idx of player, weird offsets compared to player_slot
            * player_slot: assumption -> 0-4 radiant, 128-131 dire, order not 100% clear yet
            * killer: seems to be the same as player_slot, -1 if killed by creep
            * team: 2 radiant, 3 dire
            * value: gold earned
            * key:
                + firstblood: similar to player_slot, idx of killed player
                + building: name of building e.g. npc_dota_badguys_tower1_top
        - ideas for stats (preferably stupid ones):
            * rosh kill timings radiant vs. dire
            * aegis taken vs aegis stolen
            * heroes with most firstbloods
            * first blood radiant vs. dire
            * tower kills timings distribution -> easiest/hardest tower to kill 
            * ...
    2) picks_bans:
        - keys: is_pick, hero_id, team, order, match_id
        - nothing complicated, straightforward useable
    3) radiant_gold_adv & radiant_gold_adv:
        - variable length lists -> useful for graphs and comparisons
    4) players:
        - needs a dedicated analysis, should be combinable with rest of match stats
          e.g. via match id joining
    5) chat:
        - filter chatwheel only and use the following link to get types/names
          https://github.com/odota/dotaconstants/blob/master/build/chat_wheel.json
        - keys:
            * time: ignore
            * type: after filtering, ignore
            * key: keep and use link to get useful data
            * slot: ignore
            * player_slot: ignore
        - unfortunately not all playsounds available online ...
        - last try -> local files
            * some team sounds available in game/dota/pak01_059.vpk
            * missing from top 30: 141497, 400006 (ephey), 141571 and some basic ones
            * it seems sprays are in game/dota/pak01_157.vpk -> 141497, 141571
            * hacking through vpk (valve pak) gives some nice insights about (sound)files
            * ...
        - DROP -> information retrieval for seasonal content (playsounds/sprays) too complex
            
PREPROCESSING - list objects
    1) objectives:
        - create list of rosh timings -> radiant vs. dire
        - len of list above returns rosh kills total per side
        - same idea with courier kills
        - amount of courier killed by creeps
        - amount of aegis stolen per side, should be very sparse
        - building kills -> timings for each building, killer not interesting
    2) picks_bans:
        - picks/bans radiant/dire
        - timing uninteresting, order as well (?)
        - 5*2 + 7*2 new columns -> hero_id
    3) players:
        - biggest object in dataframe
        - ...
    
            
                        

PROCESSING PIPELINE:
    1) DROP FEATURES
    2) PROCESS LIST FEATURES
        - objectives:
            * successfully extracted rosh timing for each side
            * successfully extracted courier kill timings for each side
            * successfully extractes aegis stolen + timing for each side
            * successfully extracted kill time for each building for each side
            * successfully extracted first blood timings for each side
            * finished functions
            * FINISHED PREPROCESSING
        - picks_bans:
            * extracted picks and bans for each side
            * FINISHED PREPROCESSING
            * ?potential change: dedicated columns for picks bans
        - players:
            * ...
        
        
        
        
PROBLEM #1 - opendota does not provide all chatwheel information:
    1) using python vpk and some searching leads to:
        - "steamapps/common/dota 2 beta/game/dota/pak01_dir.pak"
        - open with vpk module
        - access "scripts/chat_wheel.txt"
            * information about many sounds e.g. hero laugh and basic cmds
            * not available -> fandom: 140000-150000
        - access "scripts/chat_wheels/ti2021_casters_chat_wheel.txt"
            * 400006 is ephey laugh, 400018 is ppd
        - access to "resource/localization/teamfandom_english.txt"
            * includes entries with team fandom codes
            * also includes caster lines but without code -> sound for 400006 without actual code
              "dota_chatwheel_message_Ephey_voice" is 400006, the message is accessible in 
              "scripts/chat_wheels/ti2021_casters_chat_wheel.txt"
            * spray also available but team id is needed
            * ...
    2) conclusion:
        - some chatwheel interactions are playsounds and rarely some are sprays
        - new dota 2 system made it very complex to retrieve information about seasonal fan
          content -> playsounds and stickers are in separate files
        - playsounds are identifiable, sprays are not
        - DROP
        
        
PROBLEM #2 - player feature is huge
    1) analysis:
        - match_id: 
            * 10 players -> 10 entries with same match id
            * keep
        - player_slot: 
            * drop
        - ability_targets: 
            * dict -> {spell: {enemy_hero: amount}}
            * unreliable if compared to opendota info
            * only targeted spells
            * drop
        - ability_upgrades_arr:
            * upgrade order of spells/attributes
        - ability_uses: dictionary -> {spell: amount}
        - account_id: drop
        - actions: dictionary -> {action: amount}
            * 1: move position
            * 2: move target
            * 3: attack position
            * 4: attack target
            * 5: cast position
            * 6: cast target
            * 7: ?
            * 8: cast no target
            * 9: ?
            * 10: held position
            * 11: ?
            * rest: ?
            * undocumented feature -> opendota information helpful but not
              complete -> some features used, some note
            * keep for now
        - additional_units:
            * seems sparse e.g. arc warden clone and lone druid spirit bear -> total 3 instances
            * keeps track of items held by unit
            * keep in case an item anlysis is required
        - assists:
            * keep
        - backpack_[0-3]:
            * items in backpack at the end of the game
            * other features provide more useful item info
            * backpack_3 does not exist?
            * drop
        - buyback_log:
            * list of dicts -> time, slot, type, player_slot
            * slot and player_slot seem to be the same just different offset? dict bound
              to hero anyway?
            * every type is the same
            * keep list of times, nothing else
        - camps_stacked:
            * unsigned int -> how many camps were stacked
            * keep
        - connection_log:
            * dont care
            * drop
        - creeps_stacked:
            * amout of creep stacked -> creeps > camps
            * keep
        - damage:
            * dict of enemies and damage done -> {enemy: damage}
            * heroes, creeps (also deny damage), neutrals, hero controlled units, illusions,
              roshan, courier, buildings + probably more
            * keep
        - damage_inflictor:
            * damage dealt to heroes
            * null -> right click, spell damage, item damage, summon spell damage
            * keep
        - damage_inflictor_received:
            * damage received from enemy heroes via spells and right clicks
            * keep
        - damage_taken:
            * damage taken as sum per unit entity -> heroes, creeps, neutrals etc.
            * keep
        - damage_targets:
            * damage dealt to heroes -> dict {spell/right_click: {hero: damage}}
            * keep
        - deaths:
            * number of deaths
            * keep
        - denies:
            * number of denies
            * keep
        - dn_t:
            * denies in minute buckets -> match time == 45:10 -> 46 buckets
            * drop
        - firstblood_claimed:
            * bool -> only true for 1/10 players
            * drop
        - gold:
            * no direct information but seems to be gold at the end of the match
            * drop
        - gold_per_min:
            * good stat
            * keep
        - gold_reason:
            * dict {source: gold}
            * sources are rosh, creeps, runes etc.
            * keep
        - gold_spent:
            * number of gold spent
            * keep
        - gold_t:
            * similar to dn_t -> gold in minute buckets -> cumsum
            * keep
        - hero_damage:
            * number hero damage
            * keep
        - hero_healing:
            * number hero healing
            * keep
        - hero_hits:
            * dict {spell/rightclick: amount}
            * how many times was an enemy hit by a spell or right click
            * seems to be inconsistend or badly described -> on opendota page
              some values seem to be incorrect?
            * keep, but investigate further
        - hero_id:
            * keep
        - item_[0-5]:
            * items held at the end of the game
            * item timings seem more insightful
            * drop
        - item_neutral:
            * neutral item id at the end of the game
            * same problem as previous one
            * drop
        - item_uses:
            * dict {item_name: amount}
            * active and toggle-able items, targeted and non-targeted
            * keep
        - kill_streaks:
            * dict {kill_streak_amount: amount}
            * keep
        - killed:
            * dict {unit: amount}
            * creeps (also denies), neutrals, summons, courier, tower etc.
            * most killed npc unit etc.
            * keep
        - killed_by:
            * dict {hero/unit/building: amount}
            * hero most often killed by buildings etc.
            * keep
        - kills:
            * number amount of kills
            * keep
        - kills_log:
            * list of dicts [{time: hero}]
            * kill distribution -> early late game hero comparison
            * keep
        - lane_pos:
            * fun to play around and recreate heatmaps
            * difficult to recreate heatmap on opendota but definitely possible;
              downloading reference from webpage possible
            * keep, but learn how to properly deal with it
        - last_hits:
            * number amount of last hits
            * keep
        - leaver_status:
            * cant leave pro match
            * drop
        - level:
            * endgame level
            * keep
        - lh_t:
            * last hits in minute buckets
            * keep
        - life_state:
            * time in seconds -> alive vs. dead
            * dict {status: seconds} -> 0=alive, 1=dead, 2=dead; why two dead variables?
            * keep
        - max_hero_hit:
            * biggest amount of damage in one instance
            * seems useless
            * drop
        - multi_kills:
            * dict {multikills: amount}
            * count double kills, rampages etc.
            * keep
        - net_worth:
            * endgame networth
            * keep
        - obs:
            * similar to lane_pos
            * coordinate for observer placements
            * obs_log contains more information
            * drop
        - obs_log:
            * contains info when obs was placed -> obs_log_left contains info when destroyed
            * keep
        - obs_left_log:
            * list of dicts
                + time: when destroyed
                + key: coordinates
                + attackername: killer; if not expired -> owner killer
                + x, y, z: more detailed coordinates
            * keep
        - obs_placed:
            * obs_left_log covers this
            * drop
        - party_id:
            * useless
            * drop
        - party_size:
            * useless
            * drop
        - performance_others:
            * useless
            * drop
        - permanent_buffs:
            * list of dicts
                - buff_id -> eg. 12=shard, 6=tome, 3=int stolen etc.
                - stack count -> number
                - grant time -> seems weird -> ~11 min offset? draft time included?
                  not well documented in opendota anyway
            * not relevant enough
            * drop
        - pings:
            * number of pings
            * keep
        - pred_vict:
            * useless in pro match
            * drop
        - purchase:
            * dict {item: amount}
            * count tp purchases etc.
            * purchase_log offers the same statistics
            * drop
        - purchase_log:
            * see above
            * keep
        - randomed:
            * useless
            * drop
        - repicked:
            * useless
            * drop
        - roshans_killed:
            * bool -> did last hit rosh
            * useless
        - rune_pickups:
            * number of runes picked up, total no distinction
            * runes_log more detailed
            * drop
        - runes:
            * runes_log more detailed
            * drop
        - runes_log:
            * keep
        - sen:
            * same as obs but for sentries
            * drop
        - sen_log:
            * same as obs_log
            * keep
        - sen_left_log:
            * most info about sentries
            * keep
        - sen_placed:
            * info covered in sen_left_log
            * drop
        - stuns:
            * seconds of hero disable -> tusk stuns, ember chains, 
            * according to open dota pango has a negative value?
            * if faulty like pango negative value -> not reliable
            * drop
        - teamfight_participation:
            * percentage of participation
            * not interesting enough
            * drop
        - times:
            * list of seconds in 60 sec interval
            * same for every player
            * drop
        - tower_damage:
            * number total tower damage
            * keep for now -> maybe obsolete later
        - towers_killed:
            * number towers killed
            * keep
        - xp_per_min:
            * number xp per min
            * keep
        - xp_reasons:
            * dict {source: amount}
            * from creeps, time, hero kills etc.
            * keep
        - xp_t:
            * same as gold_t but for xp
            * keep
        - personaname:
            * ingame steam name
            * drop
        - name:
            * official dpc name
            * keep
        - last_login:
            * dont care
            * drop
        - radiant_win:
            * covered in main data frame
            * drop
        - start_time:
            * time of start match in epoch
            * dont care
            * drop
        - duration:
            * info in main df
            * drop
        - cluster:
            * apparently some server value
            * not relvant here but for replay extraction
            * drop
        - lobby_type:
            * dont care
            * drop
        - game_mode:
            * cm only
            * drop
        - is_contributor:
            * dont care
            * drop
        - patch:
            * for this purpose not important
            * drop
        - region:
            * dont care
            * drop
        - isRadiant:
            * info in main df
            * drop
        - win:
            * obsolete
            * drop
        - lose:
            * obsolete
            * drop
        - total_gold:
            * networth at the end of the match
            * per hero with different game durations no good stat
            * drop
        - total_xp:
            * same as above
            * drop
        - kda:
            * best worst player in tournament
            * keep
        - abandons:
            * irrelevant
            * drop
        - neutral_kills:
            * slayer stat
            * keep
        - tower_kills:
            * tower killer
            * same as towers_killed
            * drop
        - courier_kills:
            * courier killer
            * keep
        - lane_kills:
            * slayer stat
            * keep
        - hero_kills:
            * kills has the same information
            * drop
        - observer_kills:
            * keep
        - sentry_kills:
            * keep
        - roshan_kills:
            * keep
        - necronomicon_kills:
            * rip
            * drop
        - ancient_kills:
            * keep
        - buyback_count:
            * keep
        - observer_uses:
            * keep
        - sentry_uses:
            * keep
        - lane_efficiency:
            * percentage of lane gold (creeps+passive+start) at 10 min
            * best laner
            * keep
        - lane_efficiency_pct:
            * same as above but simplified
            * drop
        - lane:
            * number which lane
                + 1 -> safe 
                + 2 -> mid
                + 3 -> off
                + safe radiant = safe dire = 1
            * keep
        - lane_role:
            * reverse lane from above for dire heroes -> safe radiant vs. safe dire -> 1
            * safe radiant = off dire = bottom
            * drop
        - is_roaming:
            * bool
            * keep
        - purchase_time:
            * very misleading -> sum of buy times per item
            * useless if one item purchased multiple times
            * drop
        - first_purchase_time:
            * purchase log is better
            * drop
        - item_win:
            * dict {item: bool}
            * might be interesting
            * keep
        - item_usage:
            * seems wrong -> dict where each item has 1 usage?
            * drop
        - purchase_tpscroll:
            * amount of tps bought
            * keyerror if tp not bought at least once
            * drop
        - actions_per_min:
            * amount of apm
            * keep
        - life_state_dead:
            * covered in life_state
            * drop
        - rank_tier:
            * dont care
            * drop
        - is_subscriber:
            * bool if subscriber?
            * drop
        - cosmetics:
            * dont care shiny pixels
            * drop
        - benchmarks:
            * dict of various metrics compared to recent performance of played heroes
            * dont care
            * drop
            
    2) start preprocessing (only useful features):
        - ability_upgrades_arr:
            * track talent choice
            * track skill choice for each level
            * ... 
        - ability_uses:
        - actions:
        - actions_per_min:
            * simple
        - additional_units:
        - ancient_kills:
            * simple
        - assists:
            * simple
        - buyback_count:
            * simple
        - buyback_log:
        - camps_stacked:
            * simple
        - courier_kills:
            * simple
        - creeps_stacked:
            * simple
        - damage:
        - damage_inflictor:
        - damage_inflictor_received:
        - damage_taken:
            * dict with source and amount of damage
            * damage sources (unprocessed):
                + enemy heroes
                + badguys/goodguys -> creeps and towers
                + neutral: neutral creeps
                + roshan
                + fountain
            * damage sources (processed):
                + sum hero damage -> inconsistent with opendota -> summons!
                + sum creep damage
                + sum tower damage
                + sum neutral damage
                + sum roshan damage
                + sum fountain damage
            * compare with match ids and push
            * ...
        - damage_targets:
        - deaths:
            * simple
        - denies:
            * simple
        - gold_per_min:
            * simple
        - gold_reasons:
        - gold_spent:
            * simple
        - gold_t:
        - hero_damage:
            * simple
        - hero_healing:
            * simple
        - hero_hits:
        - hero_id:
            * simple
        - is_roaming:
            * simple
            * boolean
        - item_uses:
        - item_win:
        - kda:
            * simple
        - kill_streaks:
        - killed:
        - killed_by:
        - kills:
            * simple
        - kills_log:
        - lane:
            * simple
        - lane_efficiency:
            * simple
        - lane_kills:
            * simple
        - lane_pos:
        - last_hits:
            * simple
        - level:
            * simple
        - lh_t:
        - life_state:
        - match_id:
            * simple
        - multi_kills:
        - name:
            * simple
            * string
        - net_worth:
            * simple
        - neutral_kills:
            * simple
        - obs_left_log:
        - obs_log:
        - observer_kills:
            * simple
        - observer_uses:
            * simple
        - pings:
            * pings
            * ERROR: if player didnt ping -> no entry -> key error
        - purchase_log:
        - roshan_kills:
            * simple
        - runes_log:
        - sen_left_log:
        - sen_log:
        - sentry_kills:
            * simple
        - sentry_uses:
            * simple
        - tower_damage:
            * simple
        - tower_kills:
            * simple
        - xp_per_min:
            * simple
        - xp_reasons:
        - xp_t:
        
        
        
interesting links:
    - https://steamapi.xpaw.me/#IDOTA2Match_570
    - https://blog.opendota.com/2016/05/13/learnings/
    - https://github.com/skadistats/clarity
    - http://web.archive.org/web/20140922070947/http://dev.dota2.com/showthread.php?t=47115
        * includes info about official steam api -> match history not as detailed as opendota
        * explains how match replay can be retrieved -> more analysis within!