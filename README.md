Code:
#include < amxmodx >
#include < amxmisc>
#include < nvault > 
#include < engine >
#include < cstrike >
#include < hamsandwich >
#include < fakemeta >
#include < fakemeta_util >
#include < fun >
#include < csx >
#include < xs >
#include < colorchat >
#include < dhudmessage >

#define PLUGIN "Furien XP Mod"
#define VERSION "0.0.1"
#define AUTHOR "hadesownage"

#define is_ent_flare(%1) (pev(%1, pev_iuser4) == 1337) ? 1 : 0

new const szPrefix [ ] = "[Furien XP Mod]^3 -";
new Level [ 33 ], eXP [ 33 ];
new KillXp, HsXp, HeXp, KnifeXp;
new g_Menu [ 33 ];
new g_iCount[ 33 ];
new g_trail;
new amx_gamename;
#define TE_SPRITETRAIL 15
new g_damage;
new g_damages;

new gFurienXP, Vault;

new g_MaxPlayers;

new g_FuriensWin = 0;
new g_AntiFuriensWin = 0;

new gMsgScreenShake;

new g_FurienHealth;
new g_AntiFurienHealth;

#define VIP_ACCESS ADMIN_LEVEL_H

new bool:UserHaveHpAndAp [ 33 ];
new bool:UserHaveHeGrenade [ 33 ];
new bool:UserHaveGodMode [ 33 ];
new bool:UserHaveNoClip [ 33 ];
new bool:UserHaveTeleport [ 33 ];
new bool:UserHaveSuperKnife [ 33 ];
new bool:UserHaveDualMp5 [ 33 ];

new bool:UserHasChoosed [ 33 ];
new bool:g_CanUseHe [ 33 ];

new const buy_FurienHealth[] = "exhealth/zm_buyhealth.wav" 
new const buy_AntiFurienHealth[] = "exhealth/hm_buyhealth.wav" 

static const COLOR[] = "^x04"; //green
static const CONTACT[] = "/vip pentru detalii";

new maxplayers;
new gmsgSayText;
new g_ScoreAttrib;

//======NICK CHANGE-----
new const g_reason[] = "Nu este permisa schimbarea nickului pe server !";

new const g_name[] = "name";
//-------END NICK=====

//======CREDITS-----
new PlayerCredits[10000];
new SymbolsName;
//-------END CREDITS=====

//-------POWERS======

//--| Menu/Power |--//
new HasPower[33], bool:HasChose[33];
//--| HE Grenade |--//
new HE_Cooldown[33] = 0;
//--| GodMode |--//
new GodMode_Cooldown[33] = 0;
new GodMode_DurationCooldown[33] = 0;
//--| Drop Enemy Weapon |--//
new DropSprite, DropSprite2;
new Drop_Cooldown[33] = 0;
new const DROP_HIT_SND[] = "Furien/DropWpn_HIT.wav";
const WPN_NOT_DROP = ((1<<2)|(1<<CSW_HEGRENADE)|(1<<CSW_SMOKEGRENADE)|(1<<CSW_FLASHBANG)|(1<<CSW_KNIFE)|(1<<CSW_C4));
//--| Freeze |--//
new Freeze_Cooldown[33] = 0;
new FreezeSprite, FreezeSprite3;
new Frozen[33];
new Float:TempSpeed[33], Float:TempGravity[33];
new const FreezeSprite2[] = { "models/glassgibs.mdl" };
new const FROSTBREAK_SND[][] = { "Furien/FrostBreak.wav" };
new const FROSTPLAYER_SND[][] = { "Furien/FrostPlayer.wav" };
const BREAK_GLASS = 0x01;
const UNIT_SECOND = (1<<12);
const FFADE_IN = 0x0000;
//--| Drag |--//
new DRAG_MISS_SND[] = "Furien/DragMiss.wav";
new DRAG_HIT_SND[] = "Furien/DragHit.wav";
new Hooked[33], Unable2move[33], OvrDmg[33];
new Float:LastHook[33];
new bool: BindUse[33] = false, bool: Drag_I[33] = false;
new Drag_Cooldown[33] = 0;
new bool:Not_Cooldown[33];
new DragSprite;
//--| Teleport |--//
new TeleportSprite, TeleportSprite2, TeleportSprite3;
new Teleport_Cooldown[33];
new const SOUND_BLINK[] = { "weapons/flashbang-1.wav" };
const UNIT_SEC = 0x1000;
const FFADE = 0x0000;
//--| NoRecoil |--//
new Float: cl_pushangle[33][3];
const WEAPONS_BITSUM = (1<<CSW_KNIFE|1<<CSW_HEGRENADE|1<<CSW_FLASHBANG|1<<CSW_SMOKEGRENADE|1<<CSW_C4);
//--| Cvars |--//
new CvarDropDistance,
CvarDropCooldown, CvarFreezeDuration, CvarFreezeCooldown, CvarFreezeDistance, CvarDragSpeed, CvarDragCooldown,
CvarDragDmg2Stop, CvarDragUnb2Move, CvarTeleportCooldown, CvarTeleportRange;

//-------END POWERS=====

//-------SALAMANDER=====
new const fire_classname[] = "fire_salamander";
new const fire_spr_name[] = "sprites/fire_salamander.spr";

new const v_model[] = "models/furien/v_salamander.mdl";
new const p_model[] = "models/furien/p_salamander.mdl";
new const w_model[] = "models/furien/w_salamander.mdl";

new const fire_sound[] = "weapons/flamegun-2.wav";

#define CSW_SALAMANDER CSW_M249
#define PEV_ENT_TIME pev_fuser1
#define TASK_FIRE 3123123
#define TASK_RELOAD 2342342
new g_had_salamander[33], bool:is_firing[33], bool:is_reloading[33], Float:g_last_fire[33],
bool:can_fire[33], g_reload_ammo[33], g_ammo[33];

enum {
	IDLE_ANIM = 0,
	DRAW_ANIM = 4,
	RELOAD_ANIM = 3,
	SHOOT_ANIM = 1,
	SHOOT_END_ANIM = 2
}

new g_salamander;
new cvar_dmgrd_start, cvar_dmgrd_end, cvar_fire_delay, cvar_max_clip;

//-------END SALAMANDER=======

//---------FURIEN BONUS BOX======

new CvarFurienSpeed, CvarAntiFurienSpeed;
new bool:HasSpeed[33], bool:HasTeleport[33], bool:LowSpeed [ 33 ];
new const ClassName[] = "BonusBox"
new Model[2][] = {
	"models/furien/cadout_new.mdl",
	"models/furien/cadouct.mdl"
}

new Model_Yellow[2][] = {
	"models/furien/cadout_galben.mdl",
	"models/furien/cadouct_galben.mdl"
}

const UNIT_SEC = 0x1000
const FFADE = 0x0000

#define FFADE_IN		0x0000		// Just here so we don't pass 0 into the function
#define FFADE_OUT		0x0001		// Fade out (not in)
#define FFADE_MODULATE		0x0002		// Modulate (don't blend)
#define FFADE_STAYOUT		0x0004		// ignores the duration, stays faded out until new ScreenFade message received
enum {
	Red,
	Green,
	Blue
};

//---------END FURIEN BONUS BOX======

//---------K1ASUS WEAPON ( SCORPION )======

#define ENG_NULLENT		-1
#define EV_INT_WEAPONKEY	EV_INT_impulse
#define k1ases_WEAPONKEY	890
#define MAX_PLAYERS  			  32
#define IsValidUser(%1) (1 <= %1 <= g_MaxPlayers)
#define write_coord_f(%1)	engfunc(EngFunc_WriteCoord,%1)

const USE_STOPPED = 0
const OFFSET_ACTIVE_ITEM = 373
const OFFSET_WEAPONOWNER = 41
const OFFSET_LINUX = 4
const OFFSET_LINUX_WEAPONS = 4

#define WEAP_LINUX_XTRA_OFF			4
#define m_fKnown				44
#define m_flNextPrimaryAttack 			46
#define m_flTimeWeaponIdle			48
#define m_iClip					51
#define m_fInReload				54
#define PLAYER_LINUX_XTRA_OFF			4
#define m_flNextAttack				83

#define k1ases_RELOAD_TIME 2.5

new bool:k1ases_weapon [ 33 ];

new const Fire_Sounds[][] = { "weapons/k1ar-1.wav" }
new const sprites_exp[] = "sprites/deimosexp.spr"
new const explode_sound[] = "cso/deimos_skill_start.wav"

new const GUNSHOT_DECALS[] = { 41, 42, 43, 44, 45 }
new k1ases_V_MODEL[64] = "models/furien/v_k1ases.mdl"
new k1ases_P_MODEL[64] = "models/furien/p_k1ases.mdl"
new k1ases_W_MODEL[64] = "models/furien/w_k1ases.mdl"

new cvar_dmg_k1ases, cvar_recoil_k1ases, cvar_clip_k1ases, cvar_k1ases_ammo , cvar_k1asesammo , cvar_k1ases_delay , cvar_k1ases_claw , cvar_rad
new g_orig_event_k1ases, g_clip_ammo[33] , cvar_k1ases_fire
new Float:cl_pushangle_k1asus[MAX_PLAYERS + 1][3], m_iBlood[2]
new g_k1ases_TmpClip[33] , oldweap[33] ,  g_has_k1ases[33] , g_ammoclaw[33] , g_delay[33]

new sprites_exp_index

const PRIMARY_WEAPONS_BIT_SUM = (1<<CSW_SCOUT)|(1<<CSW_XM1014)|(1<<CSW_MAC10)|(1<<CSW_AUG)|(1<<CSW_UMP45)|(1<<CSW_SG550)|(1<<CSW_GALIL)|(1<<CSW_FAMAS)|(1<<CSW_AWP)|(1<<CSW_MP5NAVY)|(1<<CSW_M249)|(1<<CSW_M3)|(1<<CSW_M4A1)|(1<<CSW_TMP)|(1<<CSW_G3SG1)|(1<<CSW_SG552)|(1<<CSW_AK47)|(1<<CSW_P90)
new const WEAPONENTNAMES[][] = { "", "weapon_p228", "", "weapon_scout", "weapon_hegrenade", "weapon_xm1014", "weapon_c4", "weapon_mac10",
	"weapon_aug", "weapon_smokegrenade", "weapon_elite", "weapon_fiveseven", "weapon_ump45", "weapon_sg550",
	"weapon_galil", "weapon_famas", "weapon_usp", "weapon_glock18", "weapon_awp", "weapon_mp5navy", "weapon_m249",
	"weapon_m3", "weapon_m4a1", "weapon_tmp", "weapon_g3sg1", "weapon_flashbang", "weapon_deagle", "weapon_sg552",
"weapon_ak47", "weapon_knife", "weapon_p90" }

//---------END K1ASUS WEAPON ( SCORPION )======

//---------QUAD BARREL======

#define m_pPlayer				41
#define m_flNextPrimaryAttack		46
#define m_flNextSecondaryAttack	47
#define m_flTimeWeaponIdle		48
#define m_iClip				51
#define m_fInReload				54
#define m_fInSpecialReload		55

#define XTRA_OFS_WEAPON			4
#define XTRA_OFS_PLAYER		5
#define m_flNextAttack		83
#define m_rgAmmo_player_Slot0	376

new const qb_v_model[] = "models/furien/v_qbarrel.mdl"
new const qb_p_model[] = "models/furien/p_qbarrel.mdl"
new const qb_w_model[] = "models/furien/w_qbarrel.mdl"

new const qb_sound[5][] = {
	"weapons/qbarrel_clipin1.wav",
	"weapons/qbarrel_clipin2.wav",
	"weapons/qbarrel_clipout1.wav",
	"weapons/qbarrel_draw.wav",
	"weapons/qbarrel_shoot.wav"
}

#define CSW_QB CSW_XM1014
new g_had_qb[33], Float:g_last_fire_qb[33], Float:g_last_fire2[33], g_bloodspray, g_blood
new cvar_default_clip, cvar_delayattack, cvar_reloadtime, cvar_randmg_start, cvar_randmg_end
new g_quad_barrel

//---------END QUAD BARREL======

//---------DRAGON CANNON======

#define CSW_CANNON CSW_UMP45
#define weapon_cannon "weapon_ump45"

#define DEFAULT_W_MODEL "models/w_ump45.mdl"
#define WEAPON_SECRET_CODE 4965
#define CANNONFIRE_CLASSNAME "cannon_round"

// Fire Start
#define WEAPON_ATTACH_F 30.0
#define WEAPON_ATTACH_R 10.0
#define WEAPON_ATTACH_U -10.0

#define TASK_RESET_AMMO 5434

const pev_ammo = pev_iuser4

new const WeaponModel[3][] = {
	"models/furien/v_cannon.mdl",
	"models/furien/p_cannon.mdl",
	"models/furien/w_cannon.mdl"
}

new const WeaponSound[2][] = {
	"weapons/cannon-1.wav",
	"weapons/cannon_draw.wav"
}

new const WeaponResource[5][] = {
	"sprites/fire_cannon.spr",
	"sprites/weapon_cannon.txt",
	"sprites/640hud69.spr",
	"sprites/640hud2_cso.spr",
	"sprites/smokepuff.spr"
}

enum {
	MODEL_V = 0,
	MODEL_P,
	MODEL_W
}

enum {
	CANNON_ANIM_IDLE = 0,
	CANNON_ANIM_SHOOT1,
	CANNON_ANIM_SHOOT2,
	CANNON_ANIM_DRAW
}

new g_had_cannon[33], g_old_weapon[33], g_cannon_ammo[33], g_got_firsttime[33], Float:g_lastshot[33]
new g_cvar_defaultammo, g_cvar_reloadtime, g_cvar_firespeed, g_cvar_radiusdamage, g_cvar_damage
new Float:g_temp_reloadtime, g_smokepuff_id

//---------END DRAGON CANNON======

//---------M79 WEAPON======

#define HUD_HIDE_CROSS (1<<6)

// Weapon/Grenade models
new const m79_P_MODEL[] = "models/furien/p_m79.mdl"
new const m79_V_MODEL[] = "models/furien/v_m79fix2.mdl"
new const m79_W_MODEL[] = "models/furien/w_m79.mdl"
new const m79_GRENADE_MODEL[] = "models/grenade.mdl"

// Fire sound
new const m79_GRENADE_SHOOT[] = "weapons/m79_fire1.wav"
new const m79_GRENADE_CLIPIN[] = "weapons/m79_clipin.wav"
new const m79_GRENADE_CLIPON[] = "weapons/m79_clipon.wav"
new const m79_GRENADE_CLIPOUT[] = "weapons/m79_clipout.wav"
new const m79_GRENADE_DRAW[] = "weapons/m79_draw.wav"

// Sprites
new const m79_GRENADE_TRAIL[] = "sprites/laserbeam.spr"
new const m79_GRENADE_EXPLOSION[] = "sprites/m79_exp.spr"
new const m79_GRENADE_SMOKE[] = "sprites/black_smoke3.spr"

// Cached sprite indexes
new sTrail, sExplo, sSmoke

// Sprites
new gmsgWeaponList

// Bodyparts and blood
new mdl_gib_flesh, mdl_gib_head, mdl_gib_lung, mdl_gib_spine,
blood_drop, blood_spray

// Item ID
new m79

// Player variables
new g_hasM79[33] // whether player has M79
new g_FireM79[33] // player is shooting
new g_canShoot[33] // player can shoot
new Float:g_last_shot_time[33] // last shot time
new grenade_count[33] // current grenade count
new bool:draw_wpn[33] //Ð²Ñ‹Ð±Ð¸Ñ€Ð°ÐµÐ¼ Ð¿ÑƒÐºÐ°Ð»ÐºÑƒ
new bool:hasOnHandM79[33],bool:canfire[33],
cvar_granade_damage_radius,
cvar_granade_max_damage
// Message ID's
new g_msgScreenShake,g_msgStatusText
new gmsgDeathMsg, gmsgScoreInfo

// Customization(CHANGE HERE)
#define LAUNCHER_COST	20
new Float:delayshot = 3.0
// Tasks
#define TASK_HUDAMMO	    1337
#define TASK_FRSTSHT	    1437
#define ID_HUDAMMO (taskid - TASK_HUDAMMO)
#define ID_SHT (taskid - TASK_FRSTSHT)

enum {
	anim_idle,
	anim_shot1,
	anim_shot2,
	anim_draw,
}

//---------END M79 WEAPON======

new dual_mp5_v_model [ 66 ] = "models/furien/weapons/v_dualmp5.mdl";
new dual_mp5_p_model [ 66 ] = "models/furien/weapons/p_dualmp5.mdl";

new infinity_knife_v_model [ 66 ] = "models/furien/knifes/v_infinity_knife1.mdl";
new infinity_knife_p_model [ 66 ] = "models/furien/knifes/p_infinity_knife1.mdl";

new katana_knife_v_model [ 66 ] = "models/furien/knifes/v_katana.mdl";
new katana_knife_p_model [ 66 ] = "models/furien/knifes/p_katana.mdl";

new double_katana_v_knife_model [ 66 ] = "models/furien/knifes/v_double_katana.mdl";
new double_katana_p_knife_model [ 66 ] = "models/furien/knifes/p_double_katana.mdl";

new super_knife_v_model [ 66 ] = "models/furien/knifes/v_natad.mdl";
new super_knife_p_model [ 66 ] = "models/furien/knifes/p_natad.mdl";

new axe_knife_v_model [ 66 ] = "models/furien/knifes/v_vipaxe.mdl";
new axe_knife_p_model [ 66 ] = "models/furien/knifes/p_vipaxe.mdl";

new trainer_v_model [ 66 ] = "models/furien/knifes/v_combatknife.mdl";
new trainer_p_model [ 66 ] = "models/furien/knifes/p_combatknife.mdl";

new ignes_knife_model [ 66 ] = "models/furien/knifes/v_ignes.mdl";
new elf_knife_model [ 66 ] = "models/furien/knifes/v_elf.mdl";

new super_knife_shop_v_model [ 66 ] = "models/furien/knifes/v_superknife_shop.mdl";
// new super_knife_shop_p_model [ 66 ] = "models/furien/knifes/p_superknife_shop.mdl";

new super_knife_shop_v_model2 [ 66 ] = "models/furien/knifes/v_superknife_shop2.mdl";
new super_knife_shop_p_model2 [ 66 ] = "models/furien/knifes/p_superknife_shop2.mdl";

new thompson_v_model [ 66 ] = "models/furien/weapons/v_thompson.mdl";
new thompson_p_model [ 66 ] = "models/furien/weapons/p_thompson.mdl";

new uspx_v_model [ 66 ] = "models/furien/weapons/v_uspx.mdl";
new uspx_p_model [ 66 ] = "models/furien/weapons/p_uspx.mdl";

new hunter_v_model [ 66 ] = "models/furien/weapons/v_f2000.mdl";
new hunter_p_model [ 66 ] = "models/furien/weapons/p_f2000.mdl";

new mage_v_model [ 66 ] = "models/furien/weapons/v_fnc.mdl";
new mage_p_model [ 66 ] = "models/furien/weapons/p_fnc.mdl";

new rogue_v_model [ 66 ] = "models/furien/weapons/v_svdex.mdl";
new rogue_p_model [ 66 ] = "models/furien/weapons/p_svdex.mdl";

new shaman_v_model [ 66 ] = "models/furien/weapons/v_tar21.mdl";
new shaman_p_model [ 66 ] = "models/furien/weapons/p_tar21.mdl";

new warrior_v_model [ 66 ] = "models/furien/weapons/v_kriss.mdl";
new warrior_p_model [ 66 ] = "models/furien/weapons/p_kriss.mdl";

new deklowaz_v_model [ 66 ] = "models/furien/weapons/v_dualkriss.mdl";
new deklowaz_p_model [ 66 ] = "models/furien/weapons/p_dualkriss.mdl";

new flare_v_model [ 66 ] = "models/furien/weapons/v_flare.mdl";
new flare_w_model [ 66 ] = "models/furien/weapons/w_flare.mdl";

new strike_grenade_v_model [ 66 ] = "models/furien/weapons/v_hegrenade.mdl";
new strike_grenade_p_model [ 66 ] = "models/furien/weapons/p_hegrenade.mdl";

new bool:dual_mp5 [ 33 ];
new bool:salamander [ 33 ];
new bool:SalamanderLimit [ 33 ];
new bool:katana_knife [ 33 ];
new bool:double_katana_knife [ 33 ];
new bool:super_knife [ 33 ];
new bool:infinity_knife [ 33 ];
new bool:ignes_knife [ 33 ];
new bool:elf_knife [ 33 ];
new bool:trainer [ 33 ];
new bool:vip_axe_knife [ 33 ];
new bool:hunter [ 33 ];
new bool:mage [ 33 ];
new bool:rogue [ 33 ];
new bool:shaman [ 33 ];
new bool:warrior [ 33 ];
new bool:deklowaz [ 33 ];
new bool:thompson [ 33 ];
new bool:uspx [ 33 ];
new bool:flare [ 33 ];
new bool:druid [ 33 ];
new bool:strike_grenade [ 33 ];
new bool:strike_grenade2 [ 33 ];
new bool:strike_grenade3 [ 33 ];
new bool:super_knife_shop [ 33 ];
new bool:super_knife_shop2 [ 33 ];
new bool:UserHaveQuad [ 33 ];
new bool:UserHaveDragon [ 33 ];
new bool:UserHaveM79 [ 33 ];

new const Levels [ 30 ] =  {
	
	70, //1
	150, //2
	200, //3
	300, //4
	380, //5
	500, //6
	550, //7
	650, //8
	800, //9
	900, //10
	1000, //11
	1200, //12
	1400, //13
	1650, //14
	1800, //15
	2000, //16
	2300, //17
	2600, //18
	3000, //19
	3300, //20
	3600, //21
	4000, //22
	4300, //23
	4900, //24
	5400, //25
	6000, //26
	6500, //27
	7000, //28
	7700, //29
	8000 //30
};

new const Prefix [ 30 +2 ] [ ] = {
	
	"0",
	"1",
	"2",
	"3",
	"4",
	"5",
	"6",
	"7",
	"8",
	"9",
	"10",
	"11",
	"12",
	"13",
	"14",
	"15",
	"16",
	"17",
	"18",
	"19",
	"20",
	"21",
	"22",
	"23",
	"24",
	"25",
	"26",
	"27",
	"28",
	"29",
	"30",
	""
};

public plugin_init ( ) {
	
	register_plugin ( PLUGIN, VERSION, "hadesownage" );
	
	register_clcmd ( "say /xp", "cmdShowXp", -1 );
	register_clcmd ( "say /savexp", "cmdSaveXp", -1 );
	register_clcmd ( "say /level", "cmdShowLevel" );
	register_clcmd ( "say /levele", "cmdShowLevels", -1 );
	register_clcmd ( "say /topxp", "cmdXpTop15", -1 );
	register_clcmd ( "say /clearxp", "cmdClearXp", -1 );
	register_clcmd ( "say /xpmenu", "cmdXpMenu", -1 );
	register_clcmd ( "say /vipweapons", "cmdVipWeaponsMenu", -1 );
	register_clcmd ( "say /class", "cmdClassMenu", -1 );
	register_clcmd ( "say class", "cmdClassMenu", -1 );
	register_clcmd ( "say /refresh", "cmdRefreshXP", -1 );
	register_clcmd ( "say /shop", "cmdShop", -1 );
	register_clcmd ( "say shop", "cmdShop", -1 );
	register_clcmd ( "shop", "cmdShop", -1 );
	register_clcmd ( "say /help", "cmdHelp", -1 );
	register_clcmd ( "say /ajutor", "cmdHelp", -1 );
	register_clcmd ( "say /detalii", "cmdHelp", -1 );
	register_clcmd ( "say /despre", "cmdHelp", -1 );
	register_clcmd ( "say /vip", "cmdShowVipDetails", -1 );
	register_clcmd ( "say /depozit","Depozit", -1 );
	register_clcmd ( "say_team /depozit","Depozit", -1 );
	register_clcmd ( "say /retrage","Retrage", -1 );
	register_clcmd ( "say_team /retrage","Retrage", -1 );
	register_clcmd ( "say /credits","Show_Credits", -1 );
	register_clcmd ( "say /credite","Show_Credits", -1 );
	register_clcmd ( "say_team /credits","Show_Credits", -1 );
	register_clcmd ( "say_team /credite","Show_Credits", -1 );
	register_clcmd ( "+drag","DragStart" );
	register_clcmd ( "-drag","DragEnd" );
	register_clcmd ( "power", "Power" );
	register_clcmd ( "power2", "CmdTeleport" );
	register_clcmd ( "say /furienvip", "cmdCheckVIP", -1 );
	register_clcmd ( "say_team /furienvip", "cmdCheckVIP", -1 );
	register_clcmd ( "vippower", "VIPpower", VIP_ACCESS );
	
	register_concmd ( "amx_givexp", "cmdGiveXp", ADMIN_IMMUNITY, "<target / all> <amount>" );
	register_concmd ( "amx_setxp", "cmdSetXp", ADMIN_IMMUNITY, "<target> <amount>" );
	register_concmd ( "amx_give_credits", "Give_Credits", ADMIN_IMMUNITY, "<target / ct / t / all> <amount>" );
	register_concmd ( "amx_reset_credits", "Reset_Credits", ADMIN_IMMUNITY, "<target / ct / t / all>" );
	
	register_event ( "DeathMsg", "eDeath", "a" );
	register_event ( "DeathMsg", "Death", "a" );
	register_event ( "CurWeapon", "AntiFurienCurrentWeapon", "be", "1=1" );
	register_event ( "CurWeapon", "FurienCurrentWeapon", "be", "1=1" );
	register_event ( "HLTV", "GetRandomPlayer", "a", "1=0", "2=0" );
	//register_event ( "TextMsg", "Round_Restart", "a", "2&#Game_C", "2&#Game_w", "2&#Game_will_restart_in" );
	
	register_forward ( FM_ClientUserInfoChanged, "fwClientUserInfoChanged" );
	register_forward ( FM_PlayerPreThink, "ShowSalamanderIcon" );
	register_forward ( FM_SetModel, "fwd_setmodel" );
	register_forward ( FM_Think, "fwd_think" );
	register_forward ( FM_PlayerPreThink, "ForcePlayerSpeed" );
	register_forward ( FM_CmdStart, "CmdStart" );
	register_forward ( FM_Touch, "Touch" );
	register_forward ( FM_Touch, "Touch_Yellow" );
	register_forward ( FM_GetGameDescription, "GameDesc" ); 
	
	register_logevent ( "round_end", 2, "1=Round_End" );
	register_logevent ( "round_start", 2, "1=Round_Start" );
	
	RegisterHam ( Ham_Spawn, "player", "cmdClassMenu", 1 );
	RegisterHam ( Ham_Spawn, "player", "RefreshWeapons", 1 ); 
	RegisterHam ( Ham_TakeDamage, "player", "FurienAndAntiFurienDamage" );
	RegisterHam ( Ham_Killed, "player", "ham_player_kill" );
	RegisterHam ( Ham_Weapon_PrimaryAttack, "weapon_hegrenade", "ham_PrimaryAttack_He" );
	
	KillXp = register_cvar ( "xm_xp_pr_kill", "10" );
	HsXp = register_cvar ( "xm_xp_pr_hs", "10" ); 
	HeXp = register_cvar ( "xm_xp_pr_nade", "20" );
	KnifeXp = register_cvar ( "xm_xp_pr_knife", "25" );
	
	gFurienXP = nvault_open ( "FurienXPMod" );
	
	g_MaxPlayers = get_maxplayers ( );
	g_ScoreAttrib = get_user_msgid("ScoreAttrib");
	
	maxplayers = get_maxplayers()
	gmsgSayText = get_user_msgid("SayText")
	register_clcmd("say", "handle_say")
	register_cvar("amx_contactinfo", CONTACT, FCVAR_SERVER)
	
	SymbolsName = register_cvar ( "fr_name_symbols", "~`" ); 	//| Symbols Name Restricted |//	
	
	CvarFurienSpeed = register_cvar("amx_bonusbox_furien_speed", "1000");
	CvarAntiFurienSpeed = register_cvar("amx_bonusbox_anitfurien_speed", "750");
	
	gMsgScreenShake = get_user_msgid("ScreenShake");
	
	amx_gamename = register_cvar( "amx_gamename", "XP Mod by Hades" ); 
	
	set_task ( 30.0, "GiveBonus", 38427236, _, _, "b" );
	set_task( 1.0, "ShowHud", _, _, _, "b" );
	set_task( 120.0, "ShowMessages", _, _, _, "b" );
	//set_task( 10.0, "UpdateHudScore", _, _, _, "b" );
	//set_task( 60.0, "CheckTime", _, _, _, "b", 0 );
	
	// POWERS ------------------------------------
	register_event("DeathMsg", "Death", "a");
	RegisterHam(Ham_TakeDamage, "player", "TakeDamage");
	register_forward(FM_PlayerPreThink, "PlayerPreThink");
	
	new weapon_name[24];
	for (new i = 1; i <= 30; i++) {
		if (!(WEAPONS_BITSUM & 1 << i) && get_weaponname(i, weapon_name, 23)) {
			RegisterHam(Ham_Weapon_PrimaryAttack, weapon_name, "Weapon_PrimaryAttack_Pre");
			RegisterHam(Ham_Weapon_PrimaryAttack, weapon_name, "Weapon_PrimaryAttack_Post", 1);
		}
	}
	
	CvarDropDistance = register_cvar ("vip_drop_distance", "5000");		// Distanta maxima la care poate ajunge puterea
	CvarDropCooldown = register_cvar ("vip_drop_cooldown" , "30.0");		// Drop Enemy WPN Cooldown
	CvarFreezeDuration = register_cvar("vip_freeze_duration", "3.0");	// Freeze Duration
	CvarFreezeCooldown = register_cvar("vip_freeze_cooldown", "30.0");	// Freeze Cooldown
	CvarFreezeDistance = register_cvar ("vip_freeze_distance", "5000");	// Distanta maxima la care poate ajunge puterea
	CvarDragSpeed = register_cvar("vip_drag_speed", "500");			// Drag Speed
	CvarDragCooldown = register_cvar("vip_drag_cooldown", "15.0");		// Drag Cooldown
	CvarDragDmg2Stop = register_cvar("vip_drag_dmg2stop", "50");		// Drag Damage to stop
	CvarDragUnb2Move = register_cvar("vip_drag_unable_move", "1");		// Drag Unable to move
	CvarTeleportCooldown = register_cvar("vip_teleport_cooldown", "20.0");	// Teleport Cooldown
	CvarTeleportRange = register_cvar("vip_teleport_range", "12345");	// Teleport Range
	// POWERS ------------------------------------
	
	// SALAMANDER ------------------------------------
	
	register_event("CurWeapon", "event_curweapon", "be", "1=1");
	register_forward(FM_UpdateClientData, "fw_UpdateClientData_Post", 1);
	RegisterHam(Ham_Spawn, "player", "fw_spawn", 1);
	RegisterHam(Ham_Weapon_Reload, "weapon_m249", "fw_weapon_reload", 1);
	RegisterHam(Ham_Item_Deploy, "weapon_m249", "fw_weapon_deploy", 1);
	RegisterHam(Ham_Item_PostFrame, "weapon_m249", "fw_item_postframe", 1);
	RegisterHam(Ham_Item_AddToPlayer, "weapon_m249", "fw_item_addtoplayer", 1);
	register_forward(FM_CmdStart, "fw_cmdstart");
	register_touch(fire_classname, "*", "fw_touch");
	register_think(fire_classname, "fw_think");
	register_forward(FM_SetModel, "fw_SetModel");
	
	register_clcmd("lastinv", "check_lastinv");
	
	cvar_dmgrd_start = register_cvar("zp_salamander_dmgrandom_start", "65.0");
	cvar_dmgrd_end = register_cvar("zp_salamander_dmgrandom_end", "90.0");
	cvar_fire_delay = register_cvar("zp_salamander_fire_delay", "0.1");
	cvar_max_clip = register_cvar("zp_salamander_max_clip", "100");
	
	// SALAMANDER ------------------------------------
	
	// K1ASUS ----------------------------------------------
	register_message(get_user_msgid("DeathMsg"), "message_DeathMsg")
	register_event("CurWeapon","CurrentWeapon","be","1=1")
	RegisterHam(Ham_Item_AddToPlayer, "weapon_mp5navy", "fw_k1ases_AddToPlayer")
	RegisterHam(Ham_Use, "func_tank", "fw_UseStationary_Post", 1)
	RegisterHam(Ham_Use, "func_tankmortar", "fw_UseStationary_Post", 1)
	RegisterHam(Ham_Use, "func_tankrocket", "fw_UseStationary_Post", 1)
	RegisterHam(Ham_Use, "func_tanklaser", "fw_UseStationary_Post", 1)
	for (new i = 1; i < sizeof WEAPONENTNAMES; i++)
		if (WEAPONENTNAMES[i][0]) RegisterHam(Ham_Item_Deploy, WEAPONENTNAMES[i], "fw_Item_Deploy_Post", 1)
	RegisterHam(Ham_Weapon_PrimaryAttack, "weapon_mp5navy", "fw_k1ases_PrimaryAttack")
	RegisterHam(Ham_Weapon_PrimaryAttack, "weapon_mp5navy", "fw_k1ases_PrimaryAttack_Post", 1)
	RegisterHam(Ham_Item_PostFrame, "weapon_mp5navy", "k1ases__ItemPostFrame");
	RegisterHam(Ham_Weapon_Reload, "weapon_mp5navy", "k1ases__Reload");
	RegisterHam(Ham_Weapon_Reload, "weapon_mp5navy", "k1ases__Reload_Post", 1);
	RegisterHam(Ham_TakeDamage, "player", "fw_TakeDamage")
	register_forward(FM_SetModel, "fw_SetModel_k1asus")
	register_forward(FM_UpdateClientData, "fw_UpdateClientData_Post_k1asus", 1)
	register_forward(FM_PlaybackEvent, "fwPlaybackEvent")
	register_forward(FM_CmdStart, "fw_CmdStart")
	
	RegisterHam(Ham_TraceAttack, "worldspawn", "fw_TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "func_breakable", "fw_TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "func_wall", "fw_TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "func_door", "fw_TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "func_door_rotating", "fw_TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "func_plat", "fw_TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "func_rotating", "fw_TraceAttack", 1)
	
	cvar_dmg_k1ases = register_cvar("zp_k1ases_dmg", "1.5")
	cvar_recoil_k1ases = register_cvar("zp_k1ases_recoil", "0.5")
	cvar_clip_k1ases = register_cvar("zp_k1ases_clip", "30")
	cvar_k1ases_ammo = register_cvar("zp_k1ases_ammo", "50")
	cvar_k1asesammo =  register_cvar("zp_k1ases_clawammo", "3")
	cvar_k1ases_delay =  register_cvar("zp_k1ases_delay", "5")
	cvar_k1ases_claw = register_cvar("zp_k1ases_clawdmg", "500")
	cvar_rad =  register_cvar("zp_k1ases_clawrad", "100.0")
	cvar_k1ases_fire = register_cvar("zp_k1ases_speedfire", "0.4")
	
	// K1ASUS ----------------------------------------------
	
	// QUAD BARREL ----------------------------------------------
	
	register_forward(FM_CmdStart, "fm_cmdstart")
	register_forward(FM_UpdateClientData, "fw_UpdateClientData_Post_qb", 1)
	register_forward(FM_SetModel, "fw_SetModel_qb")	
	
	RegisterHam(Ham_TakeDamage, "player", "fw_takedmg")
	RegisterHam(Ham_TraceAttack, "worldspawn", "TraceAttack", 1)
	RegisterHam(Ham_TraceAttack, "player", "TraceAttack", 1)	
	
	RegisterHam(Ham_Weapon_Reload, "weapon_xm1014", "ham_reload", 1)
	RegisterHam(Ham_Weapon_PrimaryAttack, "weapon_xm1014", "ham_priattack", 1)
	RegisterHam(Ham_Item_PostFrame, "weapon_xm1014", "ham_postframe")
	RegisterHam(Ham_Item_AddToPlayer, "weapon_xm1014", "fw_item_addtoplayer_qb", 1)
	
	register_clcmd("lastinv", "check_draw_weapon")
	register_clcmd("slot1", "check_draw_weapon")
	
	cvar_default_clip = register_cvar("zp_qbarrel_default_clip", "4")
	cvar_delayattack = register_cvar("zp_qbarrel_delay_attack", "0.35")
	cvar_reloadtime = register_cvar("zp_qbarrel_reload_time", "3.0")
	
	cvar_randmg_start = register_cvar("zp_qbarrel_randomdmg_start", "400.0")
	cvar_randmg_end = register_cvar("zp_qbarrel_randomdmg_end", "600.0")
	
	register_event("CurWeapon", "event_curweapon_quad", "be", "1=1")
	
	// QUAD BARREL ----------------------------------------------
	
	// DRAGON CANNON ----------------------------------------------
	
	register_event("CurWeapon", "event_CurWeapon_dragon", "be", "1=1")
	
	register_forward(FM_UpdateClientData, "fw_UpdateClientData_Post_dc", 1)
	register_forward(FM_CmdStart, "fw_CmdStart_dc")
	register_forward(FM_SetModel, "fw_SetModel_dc")
	
	register_think(CANNONFIRE_CLASSNAME, "fw_Cannon_Think")
	register_touch(CANNONFIRE_CLASSNAME, "*", "fw_Cannon_Touch")
	
	RegisterHam(Ham_Spawn, "player", "fw_Spawn_Post", 1)
	RegisterHam(Ham_Item_AddToPlayer, weapon_cannon, "fw_AddToPlayer_Post", 1)
	
	g_cvar_defaultammo = register_cvar("cannon_default_ammo", "5")
	g_cvar_reloadtime = register_cvar("cannon_reload_time", "4.0")
	g_cvar_firespeed = register_cvar("cannon_fire_speed", "200.0")
	g_cvar_radiusdamage = register_cvar("cannon_radius_damage", "200.0")
	g_cvar_damage = register_cvar("cannon_damage", "700.0")
	
	register_clcmd("amx_get_dragoncannon", "get_dragoncannon", ADMIN_RCON)
	register_clcmd("weapon_cannon", "hook_weapon")
	
	// DRAGON CANNON ----------------------------------------------
	
	// M79 WEAPON ----------------------------------------------
	
	// Register new extra item
	cvar_granade_damage_radius = register_cvar("granade_damage_radius","400",FCVAR_UNLOGGED)
	cvar_granade_max_damage = register_cvar("granade_max_damage","700",FCVAR_UNLOGGED)
	
	// Events
	register_event("CurWeapon", "Event_CurrentWeapon_m79", "be", "1=1")
	register_logevent("event_start_m79", 2, "1=Round_Start")
	
	// Forwards
	RegisterHam(Ham_Killed, "player", "fw_PlayerKilled_m79")
	register_forward(FM_CmdStart, "fw_CmdStart_m79")
	register_forward(FM_PlayerPostThink, "fw_PlayerPostThink_m79")
	register_forward(FM_UpdateClientData, "fw_UpdateClientData_Post_m79", 1)
	register_clcmd("drop","dropcmd")
	// Messages
	g_msgScreenShake = get_user_msgid("ScreenShake")
	g_msgStatusText = get_user_msgid("StatusText")
	gmsgDeathMsg = get_user_msgid("DeathMsg")
	gmsgScoreInfo = get_user_msgid("ScoreInfo") 
	// Sprites
	RegisterHam( Ham_Item_AddToPlayer, "weapon_p228", "fw_AddToPlayer_m79" );
	gmsgWeaponList = get_user_msgid("WeaponList")
	
	// M79 WEAPON ----------------------------------------------
	
	
	
	server_print( "[%s] Felicitari! Detii o licenta valida, iar pluginul functioneaza perfect!", PLUGIN );
	server_print( "[%s] Pentru mai multe detalii y/m: hades.hostpixel !", PLUGIN );
	server_print( "[%s] Ip-ul Licentiat: %s, Ip-ul Serverului: %s", PLUGIN, szIp, ServerLicensedIp );
	set_task( 0.1, "SqlInit" )		
}



public plugin_end( )
	nvault_close ( gFurienXP );

stock fm_set_rendering2(entity, fx = kRenderFxNone, r = 255, g = 255, b = 255, render = kRenderNormal, amount = 16) 
{
	static Float:color[3]; color[2] = float(b), color[0] = float(r), color[1] = float(g);
	
	set_pev(entity, pev_renderfx, fx);
	set_pev(entity, pev_rendercolor, color);
	set_pev(entity, pev_rendermode,  render);
	set_pev(entity, pev_renderamt,   float(amount));
	
	return true;
}

stock set_user_scoreattrib(id, attrib = 0)
{
	message_begin(MSG_BROADCAST, g_ScoreAttrib, _, 0);
	write_byte(id);
	write_byte(attrib);
	message_end( );
}

stock Drop(id)  {
	new wpn, wpnname[32];
	wpn = get_user_weapon(id);
	if(!(WPN_NOT_DROP & (1<<wpn)) && get_weaponname(wpn, wpnname, charsmax(wpnname))) {
		engclient_cmd(id, "drop", wpnname);
	}
}

stock set_weapon_anim(id, anim)
{
	set_pev(id, pev_weaponanim, anim)
	
	message_begin(MSG_ONE_UNRELIABLE, SVC_WEAPONANIM, {0, 0, 0}, id)
	write_byte(anim)
	write_byte(pev(id, pev_body))
	message_end()
}

stock drop_weapons(id, dropwhat)
{
	static weapons[32], num, i, weaponid
	num = 0
	get_user_weapons(id, weapons, num)
	
	const PRIMARY_WEAPONS_BIT_SUM2 = (1<<CSW_SCOUT)|(1<<CSW_XM1014)|(1<<CSW_MAC10)|(1<<CSW_MAC10)|(1<<CSW_UMP45)|(1<<CSW_SG550)|(1<<CSW_MAC10)|(1<<CSW_FAMAS)|(1<<CSW_AWP)|(1<<CSW_MP5NAVY)|(1<<CSW_M249)|(1<<CSW_M3)|(1<<CSW_M4A1)|(1<<CSW_TMP)|(1<<CSW_G3SG1)|(1<<CSW_SG552)|(1<<CSW_AK47)|(1<<CSW_P90)
	
	for (i = 0; i < num; i++)
	{
		weaponid = weapons[i]
		
		if (dropwhat == 1 && ((1<<weaponid) & PRIMARY_WEAPONS_BIT_SUM2))
		{
			static wname[32]
			get_weaponname(weaponid, wname, sizeof wname - 1)
			engclient_cmd(id, "drop", wname)
		}
	}
}

stock get_position(id,Float:forw, Float:right, Float:up, Float:vStart[])
{
	new Float:vOrigin[3], Float:vAngle[3], Float:vForward[3], Float:vRight[3], Float:vUp[3]
	
	pev(id, pev_origin, vOrigin)
	pev(id, pev_view_ofs,vUp) //for player
	xs_vec_add(vOrigin,vUp,vOrigin)
	pev(id, pev_v_angle, vAngle) // if normal entity ,use pev_angles
	
	angle_vector(vAngle,ANGLEVECTOR_FORWARD,vForward) //or use EngFunc_AngleVectors
	angle_vector(vAngle,ANGLEVECTOR_RIGHT,vRight)
	angle_vector(vAngle,ANGLEVECTOR_UP,vUp)
	
	vStart[0] = vOrigin[0] + vForward[0] * forw + vRight[0] * right + vUp[0] * up
	vStart[1] = vOrigin[1] + vForward[1] * forw + vRight[1] * right + vUp[1] * up
	vStart[2] = vOrigin[2] + vForward[2] * forw + vRight[2] * right + vUp[2] * up
}

stock get_speed_vector(const Float:origin1[3],const Float:origin2[3],Float:speed, Float:new_velocity[3])
{
	new_velocity[0] = origin2[0] - origin1[0]
	new_velocity[1] = origin2[1] - origin1[1]
	new_velocity[2] = origin2[2] - origin1[2]
	new Float:num = floatsqroot(speed*speed / (new_velocity[0]*new_velocity[0] + new_velocity[1]*new_velocity[1] + new_velocity[2]*new_velocity[2]))
	new_velocity[0] *= num
	new_velocity[1] *= num
	new_velocity[2] *= num
	
	return 1;
}

stock is_player_stuck(id, Float:originF[3]) {
	engfunc(EngFunc_TraceHull, originF, originF, 0, (pev(id, pev_flags) & FL_DUCKING) ? HULL_HEAD : HULL_HUMAN, id, 0);
	
	if (get_tr2(0, TR_StartSolid) || get_tr2(0, TR_AllSolid) || !get_tr2(0, TR_InOpen))
		return true;
	
	return false;
}

stock tele_effect(const Float:torigin[3]) {
	new origin[3];
	origin[0] = floatround(torigin[0]);
	origin[1] = floatround(torigin[1]);
	origin[2] = floatround(torigin[2]);
	
	message_begin(MSG_PAS, SVC_TEMPENTITY, origin);
	write_byte(TE_BEAMCYLINDER);
	write_coord(origin[0]);
	write_coord(origin[1]);
	write_coord(origin[2]+10);
	write_coord(origin[0]);
	write_coord(origin[1]);
	write_coord(origin[2]+60);
	write_short(TeleportSprite);
	write_byte(0);
	write_byte(0);
	write_byte(3);
	write_byte(60);
	write_byte(0);
	write_byte(255);
	write_byte(255);
	write_byte(255);
	write_byte(255);
	write_byte(0);
	message_end();
}

stock tele_effect2(const Float:torigin[3]) {
	new origin[3];
	origin[0] = floatround(torigin[0]);
	origin[1] = floatround(torigin[1]);
	origin[2] = floatround(torigin[2]);
	
	message_begin(MSG_PAS, SVC_TEMPENTITY, origin);
	write_byte(TE_BEAMCYLINDER);
	write_coord(origin[0]);
	write_coord(origin[1]);
	write_coord(origin[2]+10);
	write_coord(origin[0]);
	write_coord(origin[1]);
	write_coord(origin[2]+60);
	write_short(TeleportSprite);
	write_byte(0);
	write_byte(0);
	write_byte(3);
	write_byte(60);
	write_byte(0);
	write_byte(255);
	write_byte(255);
	write_byte(255);
	write_byte(255);
	write_byte(0);
	message_end();
	
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY);
	write_byte(TE_SPRITETRAIL);
	write_coord(origin[0]);
	write_coord(origin[1]);
	write_coord(origin[2]+40);
	write_coord(origin[0]);
	write_coord(origin[1]);
	write_coord(origin[2]);
	write_short(TeleportSprite2);
	write_byte(30);
	write_byte(10);
	write_byte(1);
	write_byte(50);
	write_byte(10);
	message_end();
}

stock UTIL_CreateBeamCylinder( origin[ 3 ], addrad, sprite, startfrate, framerate, life, width, amplitude, red, green, blue, brightness, speed )
{
	message_begin( MSG_PVS, SVC_TEMPENTITY, origin ); 
	write_byte( TE_BEAMCYLINDER );
	write_coord( origin[ 0 ] );
	write_coord( origin[ 1 ] );
	write_coord( origin[ 2 ] );
	write_coord( origin[ 0 ] );
	write_coord( origin[ 1 ] );
	write_coord( origin[ 2 ] + addrad );
	write_short( sprite );
	write_byte( startfrate );
	write_byte( framerate );
	write_byte(life );
	write_byte( width );
	write_byte( amplitude );
	write_byte( red );
	write_byte( green );
	write_byte( blue );
	write_byte( brightness );
	write_byte( speed );
	message_end();
}

stock Create_TE_SPRITETRAIL3(start[3], end[3], iSprite, count, life, scale, velocity, random ){
	
	message_begin( MSG_BROADCAST,SVC_TEMPENTITY)
	write_byte( TE_SPRITETRAIL )
	write_coord( start[0] ) // start position (X)
	write_coord( start[1] ) // start position (Y)
	write_coord( start[2] + 40 ) // start position (Z)
	write_coord( end[0] ) // end position (X)
	write_coord( end[1] ) // end position (Y)
	write_coord( end[2] ) // end position (Z)
	write_short( iSprite ) // sprite index
	write_byte( count ) // count
	write_byte( life) // life in 0.1's
	write_byte( scale) // scale in 0.1's
	write_byte( velocity ) // velocity along vector in 10's
	write_byte( random ) // randomness of velocity in 10's
	message_end()
}

stock fm_cs_get_current_weapon_ent(id)
{
	return get_pdata_cbase(id, OFFSET_ACTIVE_ITEM, OFFSET_LINUX);
}

stock fm_cs_get_weapon_ent_owner(ent)
{
	return get_pdata_cbase(ent, OFFSET_WEAPONOWNER, OFFSET_LINUX_WEAPONS);
}

stock UTIL_PlayWeaponAnimation(const Player, const Sequence)
{
	set_pev(Player, pev_weaponanim, Sequence)
	
	message_begin(MSG_ONE_UNRELIABLE, SVC_WEAPONANIM, .player = Player)
	write_byte(Sequence)
	write_byte(pev(Player, pev_body))
	message_end()
}

stock play_weapon_anim(player, anim)
{
	set_pev(player, pev_weaponanim, anim)
	message_begin(MSG_ONE, SVC_WEAPONANIM, {0, 0, 0}, player)
	write_byte(anim)
	write_byte(pev(player, pev_body))
	message_end()
}

stock get_weapon_attackment(id, Float:output[3], Float:fDis = 40.0)
{ 
	new Float:vfEnd[3], viEnd[3] 
	get_user_origin(id, viEnd, 3)  
	IVecFVec(viEnd, vfEnd) 
	
	new Float:fOrigin[3], Float:fAngle[3]
	
	pev(id, pev_origin, fOrigin) 
	pev(id, pev_view_ofs, fAngle)
	
	xs_vec_add(fOrigin, fAngle, fOrigin) 
	
	new Float:fAttack[3]
	
	xs_vec_sub(vfEnd, fOrigin, fAttack)
	xs_vec_sub(vfEnd, fOrigin, fAttack) 
	
	new Float:fRate
	
	fRate = fDis / vector_length(fAttack)
	xs_vec_mul_scalar(fAttack, fRate, fAttack)
	
	xs_vec_add(fOrigin, fAttack, output)
}


//------| Save Credits |------//
public SaveCredits(id) {
	Vault = nvault_open("DepozitCredits");
	new data_credits[256], key_credits[64];
	
	new name[33];
	get_user_name(id,name,32);
	format(key_credits, 63, "%s-/", name);
	
	format(data_credits, 255, "%i#", PlayerCredits[id]);
	nvault_set(Vault, key_credits, data_credits);
	return PLUGIN_CONTINUE;
}
//------| Loading Credits |------//
public LoadCredits(id) {
	Vault = nvault_open("DepozitCredits");
	new data_credits[256], key_credits[64];
	
	new name[33];
	get_user_name(id,name,32);
	format(key_credits, 63, "%s-/", name);
	
	format(data_credits, 255, "%i#", PlayerCredits[id]);
	nvault_get(Vault, key_credits, data_credits, 255);
	replace_all(data_credits, 255, "#", " ");
	
	new Credits[32];
	parse(data_credits, Credits, 31);
	PlayerCredits[id] = str_to_num(Credits);
	return PLUGIN_CONTINUE;
} 

public SaveData ( id ) { 
	
	new szName [ 32 ];
	get_user_name ( id, szName, charsmax ( szName ) );  
	new vaultkey [ 64 ], vaultdata [ 256 ];
	
	format ( vaultkey, 63,"%s-Mod", szName ); 
	format ( vaultdata, 255,"%i#%i#",eXP [ id ],Level [ id ] ); 
	
	nvault_set ( gFurienXP, vaultkey, vaultdata ); 
	return 1; 
} 

public LoadData ( id ) { 
	
	new szName [ 32 ];
	get_user_name ( id, szName, charsmax ( szName ) ); 
	new vaultkey [ 64 ],vaultdata [ 256 ];
	
	format ( vaultkey,63,"%s-Mod", szName );
	format ( vaultdata,255,"%i#%i#", eXP [ id ], Level [ id ] ); 
	nvault_get ( gFurienXP, vaultkey, vaultdata, 255 );
	
	replace_all ( vaultdata, 255, "#", " " );
	
	new playerxp [ 32 ], playerlevel [ 32 ]; 
	
	parse ( vaultdata, playerxp, 31, playerlevel, 31 ); 
	
	eXP [ id ] = str_to_num ( playerxp );
	
	Level [ id ] = str_to_num ( playerlevel ); 
	
	return 1; 
}



public ShowHud ( id ) {
	
	if ( is_user_alive ( id ) && get_user_team ( id ) == 1 ) {
		
		if ( trainer [ id ] ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Trainer", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( infinity_knife [ id ] ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Agnos", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( super_knife [ id ] ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: XFother", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		
		if ( katana_knife [ id ] && HasPower [ id ] == 4 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Samurai | Putere: Drop Enemy Weapon", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( double_katana_knife [ id ] && HasPower [ id ] == 4 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Extra Samurai | Putere: Drop Enemy Weapon", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		
		if ( ignes_knife [ id ] && HasPower [ id ] == 5 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Ignes | Putere: Freeze Enemy", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		
		
		if ( elf_knife [ id ] && HasPower [ id ] == 5 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Elf | Putere: Freeze Enemy", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( vip_axe_knife [ id ] && HasPower [ id ] == 7 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.1 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Alcadeias | Putere: Teleport", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
	}
	
	else if ( is_user_alive ( id ) && get_user_team ( id ) == 2 ) {
		
		if ( druid [ id ] ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Druid", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( hunter [ id ] ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Hunter", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( mage [ id ] ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Mage", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		
		if ( rogue [ id ] && HasPower [ id ] == 6 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Rogue | Putere: Drag Enemy", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( shaman [ id ] && HasPower [ id ] == 6 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Shaman | Putere: Drag Enemy", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		
		if ( thompson [ id ] && HasPower [ id ] == 8 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Warlock | Putere: Norecoil", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		
		
		if ( warrior [ id ] && HasPower [ id ] == 8 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Warrior | Putere: Norecoil", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
		
		if ( deklowaz [ id ] && HasPower [ id ] == 7 ) {
			
			set_hudmessage ( 255, 255, 0, -1.0, 0.80, 0, 6.0, 1.0 );
			show_hudmessage ( id, "Viata: %d | Armura: %d | Level: %s | XP: %d | Clasa: Deklowaz | Putere: Teleport", get_user_health ( id ), get_user_armor ( id ), Prefix [ Level [ id ] ], eXP [ id ] );
			
		}
	}
}

public RemoveStuff ( id ) {
	
	remove_dragoncannon ( id );
	g_had_qb [ id ] = 0;
	dual_mp5 [ id ] = false;
	k1ases_weapon [ id ] = false;
	salamander [ id ] = false;
	SalamanderLimit [ id ] = false;
	katana_knife [ id ] = false;
	double_katana_knife [ id ] = false;
	super_knife [ id ] = false;
	infinity_knife [ id ] = false;
	elf_knife [ id ] = false;
	ignes_knife [ id ] = false;
	trainer [ id ] = false;
	thompson [ id ] = false;
	uspx [ id ] = false;
	hunter [ id ] = false;
	shaman [ id ] = false;
	UserHaveM79 [ id ] = false;
	mage [ id ] = false;
	rogue [ id ] = false;
	warrior [ id ] = false;
	deklowaz [ id ] = false;
	flare [ id ] = true;
	druid [ id ] = false;
	strike_grenade [ id ] = false;
	strike_grenade2 [ id ] = false;
	strike_grenade3 [ id ] = false;
	super_knife_shop [ id ] = false;
	super_knife_shop2 [ id ] = false;
	UserHaveQuad [ id ] = false;
	UserHaveDragon [ id ] = false;
	
	HasChose[id] = false;
	HasPower[id] = 0;
	HE_Cooldown[id] = 0;
	GodMode_Cooldown[id] = 0;
	GodMode_DurationCooldown[id] = 0;
	Drop_Cooldown[id] = 0;
	Freeze_Cooldown[id] = 0;
	Drag_Cooldown[id] = 0;
	Not_Cooldown[id] = false;
	Teleport_Cooldown[id] = 0;
	
	UserHaveHeGrenade [ id ] = false;
	UserHaveGodMode [ id ] = false;
	UserHaveSuperKnife [ id ] = false;
	UserHaveNoClip [ id ] = false;
	UserHaveHpAndAp [ id ] = false;
	UserHaveDualMp5 [ id ] = false;
	UserHasChoosed [ id ] = false;
	
}

public client_putinserver ( id ) {
	
	LoadData ( id );
	LoadCredits ( id );
	RemoveStuff ( id );
	set_task ( 3.0, "ShowHud", id, _, _, "b" );
	
	client_cmd ( id, "cl_forwardspeed 9999" );
	client_cmd ( id, "cl_backspeed 9999" );
	client_cmd ( id, "cl_sidespeed 9999" );
}

public client_disconnect ( id ) {
	
	SaveData ( id );
	SaveCredits ( id );
	RemoveStuff ( id );
	
	g_has_k1ases[id] = false
	g_delay[id] = 0
	g_ammoclaw[id] = 0
	
}


public client_connect(id) {
	
	g_has_k1ases[id] = false
	g_delay[id] = 0
	g_ammoclaw[id] = 0
	
	static sName[32];
	get_user_name(id, sName, sizeof sName - 1);
	static sChars[32];
	get_pcvar_string(SymbolsName, sChars, sizeof sChars - 1);	
	for(new i = 0 ; i < strlen(sName) ; i++) {
		for(new j = 0 ; j < strlen(sChars) ; j++) {
			if(sName[i] == sChars[j]) {
				server_cmd("kick #%d ^"Numele tau contine caractere nepermise.^"", get_user_userid(id));
				break;
			}
			else {
				LoadData ( id );
			}
		}
	}
	
	g_hasM79[id] = false
	g_canShoot[id] = false
	g_last_shot_time[id] = 0.0
	grenade_count[id] = 0
	hasOnHandM79[id] = false
	remove_icon(id)
}

public plugin_cfg ( ) {
	
	server_cmd("sv_maxspeed 99999999.0");
	server_cmd("sv_airaccelerate 99999999.0");
	
}

public plugin_natives ( ) {
	
	register_native ( "set_user_credits", "set_user_credits", 1 );
	register_native ( "get_user_credits", "get_user_credits", 1 );
	register_native ( "set_user_xp", "set_user_xp", 1 );
	register_native ( "set_user_level", "set_user_level", 1 );
	register_native ( "get_user_xp", "get_user_xp", 1 );
	register_native ( "set_user_xp", "get_user_xp", 1 );
	
	register_native("give_weapon_k1ases", "native_give_weapon_add", 1)
}

public plugin_precache ( ) {
	
	precache_model(v_model); // salamander
	precache_model(p_model); // salamander
	precache_model(w_model); // salamander
	precache_model(fire_spr_name); // salamander
	
	precache_sound(fire_sound); // salamander
	
	precache_sound("weapons/flamegun-1.wav"); // salamander
	precache_sound("weapons/flamegun_clipin1.wav"); // salamander
	precache_sound("weapons/flamegun_clipout1.wav"); // salamander
	precache_sound("weapons/flamegun_clipout2.wav"); // salamander
	precache_sound("weapons/flamegun_draw.wav"); // salamander
	
	precache_model ( katana_knife_v_model );
	precache_model ( double_katana_v_knife_model );
	precache_model ( super_knife_v_model );
	precache_model ( infinity_knife_v_model );
	precache_model ( axe_knife_v_model );
	precache_model ( ignes_knife_model );
	precache_model ( elf_knife_model );
	precache_model ( thompson_v_model );
	precache_model ( uspx_v_model );
	precache_model ( hunter_v_model );
	precache_model ( mage_v_model );
	precache_model ( rogue_v_model );
	precache_model ( shaman_v_model );
	precache_model ( warrior_v_model );
	precache_model ( deklowaz_v_model );
	precache_model ( trainer_v_model );
	precache_model ( flare_v_model );
	precache_model ( strike_grenade_v_model );
	precache_model ( super_knife_shop_v_model );
	precache_model ( super_knife_shop_v_model2 );
	
	precache_model ( katana_knife_p_model );
	precache_model ( double_katana_p_knife_model );
	precache_model ( super_knife_p_model );
	precache_model ( infinity_knife_p_model );
	precache_model ( axe_knife_p_model );
	precache_model ( strike_grenade_p_model );
	precache_model ( super_knife_shop_p_model2 );
	
	precache_model ( thompson_p_model );
	precache_model ( uspx_p_model );
	precache_model ( hunter_p_model );
	precache_model ( mage_p_model );
	precache_model ( rogue_p_model );
	precache_model ( shaman_p_model );
	precache_model ( warrior_p_model );
	precache_model ( deklowaz_p_model );
	precache_model ( trainer_p_model );
	
	precache_model ( flare_w_model );
	
	precache_model ( "models/player/furienxp/furienxp.mdl" );
	precache_model ( "models/player/furienxp2/furienxp2.mdl" );
	
	g_trail = precache_model ( "sprites/smoke.spr" );
	
	g_FurienHealth = precache_model("sprites/exhealth/health_zombie.spr") 
	g_AntiFurienHealth = precache_model("sprites/exhealth/health_human.spr") 
	precache_sound(buy_FurienHealth) 
	precache_sound(buy_AntiFurienHealth) 
	
	// POWERS
	
	precache_sound(DROP_HIT_SND);
	
	DropSprite = precache_model("sprites/lgtning.spr");
	DropSprite2 = precache_model("sprites/dropwpnexp.spr");
	
	precache_sound(DRAG_HIT_SND);
	precache_sound(DRAG_MISS_SND);
	DragSprite = precache_model("sprites/zbeam4.spr");
	
	
	new i;
	for (i = 0; i < sizeof FROSTBREAK_SND; i++)
		engfunc(EngFunc_PrecacheSound, FROSTBREAK_SND[i]);
	for (i = 0; i < sizeof FROSTPLAYER_SND; i++)
		engfunc(EngFunc_PrecacheSound, FROSTPLAYER_SND[i]);
	FreezeSprite = engfunc(EngFunc_PrecacheModel, FreezeSprite2);
	FreezeSprite3 = precache_model("sprites/laserbeam.spr");
	
	TeleportSprite = precache_model( "sprites/shockwave.spr");
	TeleportSprite2 = precache_model( "sprites/blueflare2.spr");
	TeleportSprite3 = precache_model( "sprites/teleport_start.spr");
	
	for (new i = 0; i < sizeof Model; i++)
		precache_model(Model[i])
	
	for (new i = 0; i < sizeof Model_Yellow; i++)
		precache_model(Model_Yellow[i])
	
	precache_model ( dual_mp5_v_model );
	precache_model ( dual_mp5_p_model );
	
	precache_model(k1ases_V_MODEL)
	precache_model(k1ases_P_MODEL)
	precache_model(k1ases_W_MODEL)
	precache_sound("weapons/k1ar-1.wav")
	precache_sound("weapons/k1a_clipin.wav")
	precache_sound("weapons/k1a_clipout.wav")
	precache_sound("weapons/k1a_draw.wav")
	precache_sound(explode_sound)
	m_iBlood[0] = precache_model("sprites/blood.spr")
	m_iBlood[1] = precache_model("sprites/bloodspray.spr")
	sprites_exp_index = precache_model(sprites_exp)
	precache_model("sprites/640hud5.spr")
	register_forward(FM_PrecacheEvent, "fwPrecacheEvent_Post", 1)
	
	g_damage = precache_model("sprites/furien/icon_supplybox2.spr")
	g_damages = precache_model("sprites/furien/zp_zbrespawn.spr")
	
	g_blood = precache_model("sprites/blood.spr")
	g_bloodspray = precache_model("sprites/bloodspray.spr")		
	
	precache_model(qb_v_model)
	precache_model(qb_p_model)
	precache_model(qb_w_model)
	
	for(new i = 0; i < sizeof(qb_sound); i++)
		precache_sound(qb_sound[i])
	
	
	new hades
	for(hades = 0; hades < sizeof(WeaponModel); hades++)
		engfunc(EngFunc_PrecacheModel, WeaponModel[hades])
	new ownage
	for(ownage = 0; ownage < sizeof(WeaponSound); ownage++)
		engfunc(EngFunc_PrecacheSound, WeaponSound[ownage])
	
	engfunc(EngFunc_PrecacheModel, WeaponResource[0])
	engfunc(EngFunc_PrecacheGeneric, WeaponResource[1])
	engfunc(EngFunc_PrecacheModel, WeaponResource[2])
	engfunc(EngFunc_PrecacheModel, WeaponResource[3])
	g_smokepuff_id = engfunc(EngFunc_PrecacheModel, WeaponResource[4])
	
	// Models
	precache_model(m79_P_MODEL)
	precache_model(m79_V_MODEL)
	precache_model(m79_W_MODEL)
	precache_model(m79_GRENADE_MODEL)
	
	// Sounds
	precache_sound(m79_GRENADE_SHOOT)
	precache_sound(m79_GRENADE_CLIPIN)
	precache_sound(m79_GRENADE_CLIPOUT)
	precache_sound(m79_GRENADE_CLIPON)
	precache_sound(m79_GRENADE_DRAW)
	precache_sound("weapons/357_cock1.wav")
	
	// Sprites
	sTrail = precache_model(m79_GRENADE_TRAIL)
	sExplo = precache_model(m79_GRENADE_EXPLOSION)
	sSmoke = precache_model(m79_GRENADE_SMOKE)
	
	// Bodyparts and blood
	blood_drop = precache_model("sprites/blood.spr")
	blood_spray = precache_model("sprites/bloodspray.spr")
	mdl_gib_flesh = precache_model("models/Fleshgibs.mdl")
	mdl_gib_head = precache_model("models/GIB_Skull.mdl")
	mdl_gib_lung = precache_model("models/GIB_Lung.mdl")
	mdl_gib_spine = precache_model("models/GIB_B_Bone.mdl")
	
	// Sprites
	precache_generic( "sprites/weapon_m79_sisa.txt" );
	precache_generic( "sprites/640hud42.spr" );
	precache_generic( "sprites/640hud42.spr" );
	precache_generic( "sprites/640hud7x.spr" );
	
	register_clcmd("weapon_m79_sisa", "Hook_Select")
	
}

public GameDesc( ) {
	
	static gamename[32]; 
	get_pcvar_string( amx_gamename, gamename, 31 ); 
	forward_return( FMV_STRING, gamename ); 
	return FMRES_SUPERCEDE; 
}

public cmdHelp ( id ) {
	
	show_motd ( id, "/addons/amxmodx/configs/ajutor.html" );
}

public cmdShowVipDetails ( id ) {
	
	show_motd ( id, "/addons/amxmodx/configs/vip.html" );
}

public ShowMessages ( ) {
	
	switch (random_num(1,6)) 
	{
		case 1:
		{
			ColorChat ( 0, GREEN, "%s Pentru detalii despre joc, scrieti in chat^4 /detalii^3 .^4", szPrefix );
		}
		
		case 2:
		{
			ColorChat ( 0, GREEN, "%s Daca nu va functioneaza puterea, scrieti in consola^4 bind v power^3 .^4", szPrefix );
		}
		
		case 3:
		{
			ColorChat ( 0, GREEN, "%s Pentru detalii despre VIP, scrieti in chat^4 /vip^3 .^4", szPrefix );
		}
		
		case 4:
		{
			ColorChat ( 0, GREEN, "%s Pentru a vedea vipii online, scrieti in chat^4 /vips^3 .^4", szPrefix );
		}
		
		case 5:
		{
			ColorChat ( 0, GREEN, "%s Va asteptam si pe forumul nostru,^4 www.kzh.ro^3 .^4", szPrefix );
		}
		
		case 6:
		{
			ColorChat ( 0, GREEN, "%s Daca descoperiti o eroare sau un bug, va rugam sa ne contactati .^4", szPrefix );
		}
	}
}

public set_user_credits ( id, credits ) {
	
	PlayerCredits [ id ] = credits;
	
}

public get_user_credits ( id ) {
	
	return PlayerCredits [ id ];
}

public set_user_level ( id, user_level ) {
	
	Level [ id ] = user_level;
}

public get_user_level ( id ) {
	
	return Level [ id ];
}

public set_user_xp ( id, xp ) {
	
	eXP [ id ] = xp;
	
	cmdRefreshXP2 ( id );
}

public get_user_xp ( id ) {
	
	return eXP [ id ];
	
}

public round_end ( id ) {
	
	remove_dragoncannon ( id );
	dual_mp5 [ id ] = false;
	k1ases_weapon [ id ] = false;
	salamander [ id ] = false;
	SalamanderLimit [ id ] = false;
	katana_knife [ id ] = false;
	double_katana_knife [ id ] = false;
	super_knife [ id ] = false;
	infinity_knife [ id ] = false;
	vip_axe_knife [ id ] = false;
	elf_knife [ id ] = false;
	ignes_knife [ id ] = false;
	trainer [ id ] = false;
	thompson [ id ] = false;
	uspx [ id ] = false;
	hunter [ id ] = false;
	shaman [ id ] = false;
	mage [ id ] = false;
	rogue [ id ] = false;
	warrior [ id ] = false;
	deklowaz [ id ] = false;
	druid [ id ] = false;
	strike_grenade [ id ] = false;
	strike_grenade2 [ id ] = false;
	strike_grenade3 [ id ] = false;
	HasPower[id] = 0;
	Drop_Cooldown[id] = 0;
	super_knife_shop [ id ] = false;
	super_knife_shop2 [ id ] = false;
	
	new g_iMaxPlayers = get_maxplayers ( );
	
	static Players;
	for ( Players = 1 ; Players <= g_iMaxPlayers ; Players++ )
	{
		if (!is_user_alive ( Players ) )
			continue;
		
		strip_user_weapons ( Players );
		set_pdata_int ( Players, 116, 0 );
		give_item ( Players, "weapon_knife" );
	}
	
}

public GiveBonus ( id ) {
	
	new a [ 6 ];
	
	get_time ( "%H:%M", a, 5 );
	
	if ( equal ( a, "10:00" ) || equal ( a, "12:00" ) || equal ( a, "16:00" ) || equal ( a, "20:00" ) || equal ( a, "23:00" ) ) {
		
		ColorChat ( 0, GREEN, "%s Este ora^4 bonusului^3, toti jucatorii au primit^4 5^5credite .", szPrefix );
		set_user_credits ( id, get_user_credits ( id ) + 5 );
	}
}

public round_start ( id ) {
	
	new iPlayers [ 32 ];
	new iNum;
	
	get_players ( iPlayers, iNum );
	
	for ( new i = 0; i < iNum; i++ )
	{
		g_iCount [ iPlayers [ i ] ] = 0;
		g_Menu [ iPlayers [ i ] ] = 0;
		g_had_qb [ iPlayers [ i ]] = 0;
		g_CanUseHe[iPlayers[i]] = false;
	}
	
	dual_mp5 [ id ] = false;
	k1ases_weapon [ id ] = false;
	salamander [ id ] = false;
	SalamanderLimit [ id ] = false;
	katana_knife [ id ] = false;
	double_katana_knife [ id ] = false;
	super_knife [ id ] = false;
	infinity_knife [ id ] = false;
	elf_knife [ id ] = false;
	ignes_knife [ id ] = false;
	vip_axe_knife [ id ] = false;
	trainer [ id ] = false;
	thompson [ id ] = false;
	uspx [ id ] = false;
	hunter [ id ] = false;
	shaman [ id ] = false;
	mage [ id ] = false;
	rogue [ id ] = false;
	warrior [ id ] = false;
	deklowaz [ id ] = false;
	druid [ id ] = false;
	strike_grenade [ id ] = false;
	strike_grenade2 [ id ] = false;
	strike_grenade3 [ id ] = false;
	super_knife_shop [ id ] = false;
	super_knife_shop2 [ id ] = false;
	HasPower[id] = 0;
	Drop_Cooldown[id] = 0;
	
	if ( get_user_team ( id ) == 2 ) {
		give_item ( id, "weapon_smokegrenade" );
		cs_set_user_bpammo ( id, CSW_SMOKEGRENADE, 2 );
		flare [ id ] = true;
	}
	
	new ent = FM_NULLENT
	static string_class[] = "classname"
	while ((ent = engfunc(EngFunc_FindEntityByString, ent, string_class, ClassName))) 
		set_pev(ent, pev_flags, FL_KILLME)
	
	for(new id = 1; id < get_maxplayers();id++) {
		HasSpeed[id] = false
		HasTeleport[id] = false	
	}
}

public ham_PrimaryAttack_He ( iEnt ) {
	
	new id = pev( iEnt, pev_owner ); 
	
	if( g_CanUseHe [ id ] ) 
	{
		
		set_hudmessage( 0, 100, 200, -1.0, 0.35, 1, 0.01, 3.0, 1.0, 1.0 ); 
		show_hudmessage( id, "Bomba a fost plantata, nu mai poti folosi HE-urile" ); 
		
		return HAM_SUPERCEDE; 
	} 
	
	return HAM_IGNORED; 
	
}

public EventHLTV ( ) {
	
	set_task ( 0.1, "GiveBonus" );
}

public native_give_weapon_add(id)
{
	give_k1ases(id)
}

public fwPrecacheEvent_Post(type, const name[])
{
	if (equal("events/mp5n.sc", name))
	{
		g_orig_event_k1ases = get_orig_retval()
		return FMRES_HANDLED
	}
	
	return FMRES_IGNORED
}

public fw_TraceAttack(iEnt, iAttacker, Float:flDamage, Float:fDir[3], pentru, iDamageType)
{
	if(!is_user_alive(iAttacker))
		return;
	
	new g_currentweapon = get_user_weapon(iAttacker)
	if(g_currentweapon != CSW_MP5NAVY) return
	
	if((g_currentweapon == CSW_MP5NAVY && !g_has_k1ases[iAttacker])) return
	
	static Float:flEnd[3]
	get_tr2(pentru, TR_vecEndPos, flEnd)
	
	if(iEnt)
	{
		// Put decal on an entity
		message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
		write_byte(TE_DECAL)
		write_coord_f(flEnd[0])
		write_coord_f(flEnd[1])
		write_coord_f(flEnd[2])
		write_byte(GUNSHOT_DECALS[random_num ( 0, sizeof GUNSHOT_DECALS -1 ) ] )
		write_short(iEnt)
		message_end()
	}
	else
	{
		// Put decal on "world" (a wall)
		message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
		write_byte(TE_WORLDDECAL)
		write_coord_f(flEnd[0])
		write_coord_f(flEnd[1])
		write_coord_f(flEnd[2])
		write_byte(GUNSHOT_DECALS[random_num ( 0, sizeof GUNSHOT_DECALS -1 ) ] )
		message_end()
	}
	
	// Show sparcles
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
	write_byte(TE_GUNSHOTDECAL)
	write_coord_f(flEnd[0])
	write_coord_f(flEnd[1])
	write_coord_f(flEnd[2])
	write_short(iAttacker)
	write_byte(GUNSHOT_DECALS[random_num ( 0, sizeof GUNSHOT_DECALS -1 ) ] )
	message_end()
}

public fwClientUserInfoChanged(id, buffer)
{
	if (!is_user_connected(id))
		return FMRES_IGNORED;
	
	static name[32], val[32]
	get_user_name(id, name, sizeof name - 1)
	engfunc(EngFunc_InfoKeyValue, buffer, g_name, val, sizeof val - 1)
	if (equal(val, name))
		return FMRES_IGNORED;
	
	engfunc(EngFunc_SetClientKeyValue, id, buffer, g_name, name)
	console_print ( id, "%s", g_reason );
	
	return FMRES_SUPERCEDE;
}

public SalamanderGiveItem(id, itemid)
{
	if(itemid == g_salamander)
	{
		g_had_salamander[id] = true
		is_reloading[id] = false
		is_firing[id] = false
		can_fire[id] = true
		
		fm_give_item(id, "weapon_m249")
		g_ammo[id] = 30
		cs_set_user_bpammo ( id, CSW_M249, 20 );
	}
}

public get_dragoncannon(id)
{
	if(!is_user_alive(id))
		return
	
	drop_weapons(id, 1)
	
	g_had_cannon[id] = 1
	g_cannon_ammo[id] = get_pcvar_num(g_cvar_defaultammo)
	fm_give_item(id, weapon_cannon)
}

public remove_dragoncannon(id)
{
	if(!is_user_connected(id))
		return
	
	g_had_cannon[id] = 0
	g_got_firsttime[id] = 0
	g_cannon_ammo[id] = 0
	
	remove_task(id+TASK_RESET_AMMO)
}

public hook_weapon(id) engclient_cmd(id, weapon_cannon)

public event_CurWeapon_dragon(id)
{
	if(!is_user_alive(id))
		return
	
	if(get_user_weapon(id) == CSW_CANNON && g_had_cannon[id])
	{
		if(!g_got_firsttime[id])
		{
			static cannon_weapon
			cannon_weapon = fm_find_ent_by_owner(-1, weapon_cannon, id)
			
			if(pev_valid(cannon_weapon)) cs_set_weapon_ammo(cannon_weapon, 25)
			g_got_firsttime[id] = 1
		}
		
		set_pev(id, pev_viewmodel2, WeaponModel[MODEL_V])
		set_pev(id, pev_weaponmodel2, WeaponModel[MODEL_P])
		
		if(g_old_weapon[id] != CSW_CANNON)
		{
			g_temp_reloadtime = get_pcvar_float(g_cvar_reloadtime)
			set_weapon_anim(id, CANNON_ANIM_DRAW)
		}
		
		update_ammo(id)
	}
	
	g_old_weapon[id] = get_user_weapon(id)
}


public dragoncannon_shoothandle(id)
{
	if(pev(id, pev_weaponanim) != CANNON_ANIM_IDLE)
		return
	
	if(get_gametime() - g_temp_reloadtime > g_lastshot[id])
	{
		dragoncannon_shootnow(id)
		g_lastshot[id] = get_gametime()
	}
}

public dragoncannon_shootnow(id)
{
	if(g_cannon_ammo[id] == 1)
	{
		set_task(0.5, "set_weapon_outofammo", id+TASK_RESET_AMMO)
	}
	if(g_cannon_ammo[id] <= 0)
	{
		return
	}
	
	create_fake_attack(id)
	
	g_cannon_ammo[id]--
	
	set_weapon_anim(id, random_num(CANNON_ANIM_SHOOT1, CANNON_ANIM_SHOOT2))
	emit_sound(id, CHAN_WEAPON, WeaponSound[0], 1.0, ATTN_NORM, 0, PITCH_NORM)
	
	set_player_nextattack(id, CSW_CANNON, g_temp_reloadtime)
	update_ammo(id)
	
	make_fire_effect(id)
	make_fire_smoke(id)
	check_radius_damage(id)
}

public create_fake_attack(id)
{
	static cannon_weapon
	cannon_weapon = fm_find_ent_by_owner(-1, "weapon_knife", id)
	
	if(pev_valid(cannon_weapon)) ExecuteHam(Ham_Weapon_PrimaryAttack, cannon_weapon)	
}

public set_weapon_outofammo(id)
{
	id -= TASK_RESET_AMMO
	if(!is_user_alive(id))
		return
	if(get_user_weapon(id) != CSW_CANNON || !g_had_cannon[id])
		return
	
	set_weapon_anim(id, CANNON_ANIM_IDLE)
}

public make_fire_effect(id)
{
	const MAX_FIRE = 10
	static Float:Origin[MAX_FIRE][3]
	
	// Stage 1
	get_position(id, 30.0, 50.0, WEAPON_ATTACH_U, Origin[0])
	get_position(id, 30.0, 40.0, WEAPON_ATTACH_U, Origin[1])
	get_position(id, 30.0, -40.0, WEAPON_ATTACH_U, Origin[2])
	get_position(id, 30.0, -50.0, WEAPON_ATTACH_U, Origin[2])
	
	// Stage 2
	get_position(id, 50.0, 30.0, WEAPON_ATTACH_U, Origin[3])
	get_position(id, 50.0, 0.0, WEAPON_ATTACH_U, Origin[4])
	get_position(id, 50.0, -30.0, WEAPON_ATTACH_U, Origin[5])
	
	// Stage 3
	get_position(id, 70.0, 20.0, WEAPON_ATTACH_U, Origin[3])
	get_position(id, 70.0, -20.0, WEAPON_ATTACH_U, Origin[5])	
	
	// Stage 4
	get_position(id, 90.0, 0.0, WEAPON_ATTACH_U, Origin[4])
	
	for(new i = 0; i < MAX_FIRE; i++)
		create_fire(id, Origin[i])
}

public create_fire(id, Float:Origin[3])
{
	new iEnt = create_entity("env_sprite")
	static Float:vfAngle[3], Float:MyOrigin[3], Float:TargetOrigin[3], Float:Velocity[3]
	
	pev(id, pev_angles, vfAngle)
	pev(id, pev_origin, MyOrigin)
	
	vfAngle[2] = float(random(18) * 20)
	
	// set info for ent
	set_pev(iEnt, pev_movetype, MOVETYPE_PUSHSTEP)
	set_pev(iEnt, pev_rendermode, kRenderTransAdd)
	set_pev(iEnt, pev_renderamt, 250.0)
	set_pev(iEnt, pev_fuser1, get_gametime() + 2.5)	// time remove
	set_pev(iEnt, pev_scale, 2.0)
	set_pev(iEnt, pev_nextthink, halflife_time() + 0.05)
	
	entity_set_string(iEnt, EV_SZ_classname, CANNONFIRE_CLASSNAME)
	engfunc(EngFunc_SetModel, iEnt, WeaponResource[0])
	set_pev(iEnt, pev_mins, Float:{-5.0, -5.0, -5.0})
	set_pev(iEnt, pev_maxs, Float:{5.0, 5.0, 5.0})
	set_pev(iEnt, pev_origin, Origin)
	set_pev(iEnt, pev_gravity, 0.01)
	set_pev(iEnt, pev_angles, vfAngle)
	set_pev(iEnt, pev_solid, 1)
	set_pev(iEnt, pev_owner, id)	
	set_pev(iEnt, pev_frame, 0.0)
	
	// Set Velocity
	get_position(id, 100.0, 0.0, -5.0, TargetOrigin)
	
	get_speed_vector(MyOrigin, TargetOrigin, get_pcvar_float(g_cvar_firespeed), Velocity)
	set_pev(iEnt, pev_velocity, Velocity)
}

public fw_Cannon_Think(iEnt)
{
	if(!pev_valid(iEnt)) 
		return
	
	new Float:fFrame, Float:fNextThink
	pev(iEnt, pev_frame, fFrame)
	
	// effect exp
	new iMoveType = pev(iEnt, pev_movetype)
	if (iMoveType == MOVETYPE_NONE)
	{
		fNextThink = 0.0015
		fFrame += 0.5
		
		if (fFrame > 21.0)
		{
			engfunc(EngFunc_RemoveEntity, iEnt)
			return
		}
	}
	
	// effect normal
	else
	{
		fNextThink = 0.045
		fFrame += 0.5
		fFrame = floatmin(21.0, fFrame)
	}
	
	set_pev(iEnt, pev_frame, fFrame)
	set_pev(iEnt, pev_nextthink, halflife_time() + fNextThink)
	
	// time remove
	new Float:fTimeRemove
	pev(iEnt, pev_fuser1, fTimeRemove)
	if (get_gametime() >= fTimeRemove)
	{
		engfunc(EngFunc_RemoveEntity, iEnt)
		return;
	}
}

public fw_Cannon_Touch(ent, id)
{
	if(!pev_valid(ent))
		return
	
	set_pev(ent, pev_movetype, MOVETYPE_NONE)
	set_pev(ent, pev_solid, SOLID_NOT)
}

public make_fire_smoke(id)
{
	static Float:Origin[3]
	get_position(id, WEAPON_ATTACH_F, WEAPON_ATTACH_R, WEAPON_ATTACH_U, Origin)
	
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY) 
	write_byte(TE_EXPLOSION) 
	engfunc(EngFunc_WriteCoord, Origin[0])
	engfunc(EngFunc_WriteCoord, Origin[1])
	engfunc(EngFunc_WriteCoord, Origin[2])
	write_short(g_smokepuff_id) 
	write_byte(10)
	write_byte(30)
	write_byte(14)
	message_end()
}

public update_ammo(id)
{
	if(!is_user_alive(id))
		return
	
	message_begin(MSG_ONE_UNRELIABLE, get_user_msgid("CurWeapon"), _, id)
	write_byte(1)
	write_byte(CSW_CANNON)
	write_byte(-1)
	message_end()
	
	message_begin(MSG_ONE_UNRELIABLE, get_user_msgid("AmmoX"), _, id)
	write_byte(6)
	write_byte(g_cannon_ammo[id])
	message_end()
}

public check_radius_damage(id)
{
	static Float:Origin[3]
	for(new i = 0; i < get_maxplayers(); i++)
	{
		if(!is_user_alive(i))
			continue
		if(cs_get_user_team(id) == cs_get_user_team(i))
			continue
		if(id == i)
			continue
		pev(i, pev_origin, Origin)
		if(!is_in_viewcone(id, Origin, 1))
			continue
		if(entity_range(id, i) >= get_pcvar_float(g_cvar_radiusdamage))
			continue
		
		ExecuteHamB(Ham_TakeDamage, i, 0, id, get_pcvar_float(g_cvar_damage), DMG_BURN)
	}
}

public fw_UpdateClientData_Post_dc(id, sendweapons, cd_handle)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return FMRES_IGNORED
	if(get_user_weapon(id) != CSW_CANNON || !g_had_cannon[id])
		return FMRES_IGNORED
	
	set_cd(cd_handle, CD_flNextAttack, get_gametime() + 0.001) 
	
	return FMRES_HANDLED
}

public fw_CmdStart_dc(id, uc_handle, seed)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return FMRES_IGNORED
	if(get_user_weapon(id) != CSW_CANNON || !g_had_cannon[id])
		return FMRES_IGNORED
	
	static CurButton
	CurButton = get_uc(uc_handle, UC_Buttons)
	
	if(CurButton & IN_ATTACK)
	{
		CurButton &= ~IN_ATTACK
		set_uc(uc_handle, UC_Buttons, CurButton)
		
		dragoncannon_shoothandle(id)
	}
	
	return FMRES_HANDLED
}

public fw_SetModel_dc(entity, model[])
{
	if(!pev_valid(entity))
		return FMRES_IGNORED
	
	static szClassName[33]
	pev(entity, pev_classname, szClassName, charsmax(szClassName))
	
	if(!equal(szClassName, "weaponbox"))
		return FMRES_IGNORED
	
	static id
	id = pev(entity, pev_owner)
	
	if(equal(model, DEFAULT_W_MODEL))
	{
		static weapon
		weapon = fm_find_ent_by_owner(-1, weapon_cannon, entity)
		
		if(!pev_valid(weapon))
			return FMRES_IGNORED
		
		if(g_had_cannon[id])
		{
			set_pev(weapon, pev_impulse, WEAPON_SECRET_CODE)
			set_pev(weapon, pev_ammo, g_cannon_ammo[id])
			
			engfunc(EngFunc_SetModel, entity, WeaponModel[MODEL_W])
			remove_dragoncannon(id)
			
			return FMRES_SUPERCEDE
		}
	}
	
	return FMRES_IGNORED
}

public fw_Spawn_Post(id)
{
	remove_dragoncannon(id)
}

public fw_AddToPlayer_Post(ent, id)
{
	if(!pev_valid(ent))
		return HAM_IGNORED
	
	if(pev(ent, pev_impulse) == WEAPON_SECRET_CODE)
	{
		remove_dragoncannon(id)
		
		g_had_cannon[id] = 1
		g_got_firsttime[id] = 0
		g_cannon_ammo[id] = pev(ent, pev_ammo)
	}
	
	message_begin(MSG_ONE_UNRELIABLE, get_user_msgid("WeaponList"), _, id)
	write_string(g_had_cannon[id] == 1 ? "weapon_cannon" : "weapon_ump45")
	write_byte(6)
	write_byte(20)
	write_byte(-1)
	write_byte(-1)
	write_byte(0)
	write_byte(15)
	write_byte(CSW_CANNON)
	write_byte(0)
	message_end()			
	
	return HAM_HANDLED	
}

public fw_item_postframe(ent)
{
	if(!is_valid_ent(ent))
		return HAM_IGNORED
	
	static id
	id = pev(ent, pev_owner)
	
	if(!is_user_alive(id) || !is_user_connected(id))
		return HAM_IGNORED
	
	if(get_user_weapon(id) != CSW_SALAMANDER || !g_had_salamander[id])
		return HAM_IGNORED
	
	if(!is_reloading[id])
	{
		static iAnim
		iAnim = pev(id, pev_weaponanim)
		
		if(iAnim == RELOAD_ANIM)
			play_weapon_anim(id, IDLE_ANIM)
	}
	
	static salamander
	salamander = fm_find_ent_by_class(-1, "weapon_m249")
	
	set_pdata_int(salamander, 54, 0, 4)
	
	return HAM_HANDLED
}

public fw_item_addtoplayer(ent, id)
{
	if(!is_valid_ent(ent))
		return HAM_IGNORED
	
	if(entity_get_int(ent, EV_INT_impulse) == 701)
	{
		g_had_salamander[id] = true
		g_ammo[id] = pev(ent, pev_iuser3)
		entity_set_int(id, EV_INT_impulse, 0)
		
		play_weapon_anim(id, DRAW_ANIM)
		set_task(1.0, "make_wpn_canfire", id)
		
		return HAM_HANDLED
	}		
	
	return HAM_HANDLED
}

public Round_Restart ( ) {
	
	g_FuriensWin = 0;
	g_AntiFuriensWin = 0;
}

public UpdateHudScore ( ) { 
	
	set_dhudmessage ( 0, 100, 200, -1.0, 0.04, 0, 6.0, 10.1 );
	show_dhudmessage ( 0, "[ FR ] - vs - [ AF ]" );
	
	set_hudmessage ( 213, 0, 0, -1.0, 0.07, 0, 6.0, 10.1 );
	show_hudmessage ( 0, "%d - %d", g_FuriensWin, g_AntiFuriensWin );
} 

public check_lastinv(id)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return PLUGIN_HANDLED
	
	if(get_user_weapon(id) == CSW_SALAMANDER && g_had_salamander[id])
	{
		set_task(0.5, "start_check_draw", id)
	}
	
	return PLUGIN_CONTINUE
}

public start_check_draw(id)
{
	if(can_fire[id])
		can_fire[id] = false
}

public event_curweapon(id)
{
	if(is_user_alive(id) && get_user_weapon(id) == CSW_SALAMANDER && g_had_salamander[id] )
	{
		set_pev(id, pev_viewmodel2, v_model)
		set_pev(id, pev_weaponmodel2, p_model)
	}
}

public fw_weapon_deploy(ent)
{
	static id
	id = pev(ent, pev_owner)
	
	if(!is_user_alive(id) || !is_user_connected(id) )
		return HAM_IGNORED
	
	if(!g_had_salamander[id])
		return HAM_IGNORED
	
	can_fire[id] = false
	
	play_weapon_anim(id, DRAW_ANIM)
	set_task(1.0, "make_wpn_canfire", id)
	
	return HAM_HANDLED
}

public make_wpn_canfire(id)
{
	can_fire[id] = true
}

public fw_weapon_reload(ent)
{
	static id
	id = pev(ent, pev_owner)
	
	if(!is_user_alive(id) || !is_user_connected(id) )
		return HAM_IGNORED
	
	if(get_user_weapon(id) != CSW_SALAMANDER && !g_had_salamander[id])
		return HAM_IGNORED
	
	return HAM_SUPERCEDE
}

public client_PostThink(id)
{
	if(is_user_alive(id) && is_user_connected(id))
	{
		if(g_had_salamander[id] && get_user_weapon(id) != CSW_SALAMANDER)
		{
			if(can_fire[id])
				can_fire[id] = false
			
			if(is_reloading[id])
			{
				is_reloading[id] = false
				if(task_exists(id+TASK_RELOAD)) remove_task(id+TASK_RELOAD)
			}			
			} else if(g_had_salamander[id] && get_user_weapon(id) == CSW_SALAMANDER) {
			static salamander
			salamander = fm_get_user_weapon_entity(id, CSW_M249)
			
			cs_set_weapon_ammo(salamander, g_ammo[id])
		}
	}
	
}

public fw_SetModel(entity, model[])
{
	if(!is_valid_ent(entity))
		return FMRES_IGNORED;
	
	static szClassName[33]
	entity_get_string(entity, EV_SZ_classname, szClassName, charsmax(szClassName))
	
	if(!equal(szClassName, "weaponbox"))
		return FMRES_IGNORED;
	
	static iOwner
	iOwner = entity_get_edict(entity, EV_ENT_owner)
	
	if(equal(model, "models/w_m249.mdl"))
	{
		static iStoredAugID
		iStoredAugID = find_ent_by_owner(-1, "weapon_m249", entity)
		
		if(!is_valid_ent(iStoredAugID))
			return FMRES_IGNORED;
		
		if(g_had_salamander[iOwner])
		{
			entity_set_int(iStoredAugID, EV_INT_impulse, 701)
			g_had_salamander[iOwner] = false
			set_pev(iStoredAugID, pev_iuser3, g_ammo[iOwner])
			entity_set_model(entity, w_model)
			
			return FMRES_SUPERCEDE;
		}
	}
	
	return FMRES_IGNORED;
}

public fw_UpdateClientData_Post(id, sendweapons, cd_handle)
{
	if(!is_user_alive(id) || !is_user_connected(id) )
		return FMRES_IGNORED
	
	if(get_user_weapon(id) != CSW_SALAMANDER || !g_had_salamander[id])
		return FMRES_IGNORED
	
	set_cd(cd_handle, CD_flNextAttack, halflife_time() + 0.001)  
	
	return FMRES_HANDLED
}

public fw_cmdstart(id, uc_handle, seed)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return FMRES_IGNORED
	
	if(get_user_weapon(id) != CSW_SALAMANDER || !g_had_salamander[id])
		return FMRES_IGNORED
	
	static Button
	Button = get_uc(uc_handle, UC_Buttons)
	
	if(Button & IN_ATTACK)
	{
		if((get_gametime() - get_pcvar_float(cvar_fire_delay) > g_last_fire[id]))
		{
			if(can_fire[id] && !is_reloading[id])
			{
				if(g_ammo[id] > 0)
				{
					if(pev(id, pev_weaponanim) != SHOOT_ANIM)
						play_weapon_anim(id, SHOOT_ANIM)
					
					if(task_exists(id+TASK_FIRE)) remove_task(id+TASK_FIRE)
					is_firing[id] = true
					throw_fire(id)
					emit_sound(id, CHAN_WEAPON, "weapons/flamegun-2.wav", 1.0, ATTN_NORM, 0, PITCH_NORM)
					g_ammo[id]--
				}
				
			}
			g_last_fire[id] = get_gametime()
		}
		} else {
		if(is_firing[id])
		{
			if(!task_exists(id+TASK_FIRE))
			{
				set_task(0.1, "stop_fire", id+TASK_FIRE)
				emit_sound(id, CHAN_WEAPON, "weapons/flamegun-2.wav", 1.0, ATTN_NORM, 0, PITCH_NORM)
			}
		}
		
	}
	
	if(Button & IN_RELOAD)
	{
		if(!is_reloading[id] && !is_firing[id])
		{
			static curammo, require_ammo, bpammo
			
			curammo = g_ammo[id]
			bpammo = cs_get_user_bpammo(id, CSW_SALAMANDER)
			require_ammo = get_pcvar_num(cvar_max_clip) - curammo
			
			if(bpammo > require_ammo)
			{
				g_reload_ammo[id] = require_ammo
				} else {
				g_reload_ammo[id] = bpammo
			}
			
			if(g_ammo[id] < get_pcvar_num(cvar_max_clip) && bpammo > 0)
			{
				is_reloading[id] = true
				play_weapon_anim(id, RELOAD_ANIM)
				
				set_task(5.0, "finish_reload", id+TASK_RELOAD)
			}
		}
	}
	
	Button &= ~IN_ATTACK
	set_uc(uc_handle, UC_Buttons, Button)
	
	Button &= ~IN_RELOAD
	set_uc(uc_handle, UC_Buttons, Button)
	
	return FMRES_HANDLED
}

public ham_player_kill(victim, attacker, corpse, args[])
{
	
	
	
	if ( get_user_team ( victim ) == 1 ) {
		
		new vVictimOrigin[3], vAttackerorigin[3]; 
		get_user_origin( victim, vVictimOrigin ); 
		get_user_origin( attacker, vAttackerorigin ); 
		
		switch (random_num(0, 2))
		{
			case 0:
			{
				
				message_begin(MSG_ONE,SVC_TEMPENTITY,{0,0,0},attacker)
				
				write_byte(TE_SPRITETRAIL)
				write_coord(vAttackerorigin[0]) 
				write_coord(vAttackerorigin[1]) 
				write_coord(vAttackerorigin[2]) 
				write_coord(vVictimOrigin[0]) 
				write_coord(vVictimOrigin[1]) 
				write_coord(vVictimOrigin[2]) 
				write_short(g_damage) 
				write_byte(5) 
				write_byte(15) 
				write_byte(3) 
				write_byte(2) 
				write_byte(6) 
				message_end()
			}
			case 1:
			{
				message_begin(MSG_ONE,SVC_TEMPENTITY,{0,0,0},attacker)
				
				write_byte(TE_SPRITETRAIL)
				write_coord(vAttackerorigin[0]) 
				write_coord(vAttackerorigin[1]) 
				write_coord(vAttackerorigin[2]) 
				write_coord(vVictimOrigin[0]) 
				write_coord(vVictimOrigin[1]) 
				write_coord(vVictimOrigin[2]) 
				write_short(g_damages) 
				write_byte(5) 
				write_byte(15) 
				write_byte(3) 
				write_byte(2) 
				write_byte(6) 
				message_end()
			}
		}	
	}
}

public fw_CmdStart(id, uc_handle, seed)
{
	if(id > 32)
		return PLUGIN_HANDLED
	
	if(!is_user_alive(id) ) 
		return PLUGIN_HANDLED
	
	if((get_uc(uc_handle, UC_Buttons) & IN_ATTACK2) && !(pev(id, pev_oldbuttons) & IN_ATTACK2))
	{
		new szClip, szAmmo
		new szWeapID = get_user_weapon(id, szClip, szAmmo)
		if(szWeapID == CSW_MP5NAVY && g_has_k1ases[id])
		{
			weapon_ability(id)
		}
		
	}
	
	return PLUGIN_HANDLED
}

public weapon_ability(id)
{
	if(!is_user_alive(id) || g_ammoclaw[id] <= 0 || g_delay[id])
		return;
	
	set_pdata_float(id, m_flNextAttack, 1.0, PLAYER_LINUX_XTRA_OFF)
	UTIL_PlayWeaponAnimation(id, 6)
	
	new aimOrigin[3]
	get_user_origin(id, aimOrigin, 3)
	
	message_begin(MSG_BROADCAST,SVC_TEMPENTITY); 
	write_byte(TE_EXPLOSION); // TE_EXPLOSION
	write_coord(aimOrigin[0]); // origin x
	write_coord(aimOrigin[1]); // origin y
	write_coord(aimOrigin[2]); // origin z
	write_short(sprites_exp_index); // sprites
	write_byte(40); // scale in 0.1's
	write_byte(30); // framerate
	write_byte(14); // flags 
	message_end(); // message end
	
	
	new Float:aimOrigin2[3]
	
	static iVictim
	iVictim = -1
	
	aimOrigin2[0] = float(aimOrigin[0])
	aimOrigin2[1] = float(aimOrigin[1])
	aimOrigin2[2] = float(aimOrigin[2])
	
	while((iVictim = find_ent_in_sphere(iVictim, aimOrigin2, get_pcvar_float(cvar_rad))) != 0)
	{
		// Non-player entity
		if(is_user_connected(iVictim))
		{
			if(is_user_alive(iVictim)) radius_damage1(id,iVictim)
		}
	}
	
	g_ammoclaw[id] -= 1
	g_delay[id] = 1
	set_task(float(get_pcvar_num(cvar_k1ases_delay)),"can_use",id)
	client_print(id,print_center,"[Claw Ammo: %d]",g_ammoclaw[id])
	emit_sound(id, CHAN_WEAPON,explode_sound, VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
	
}
public radius_damage1(iAttacker, iVictim)
{
	ExecuteHamB(Ham_TakeDamage, iVictim , iAttacker , iAttacker, get_pcvar_float(cvar_k1ases_claw), DMG_BULLET);
}
public can_use(id)
{
	g_delay[id] = 0
}
public fw_SetModel_k1asus(entity, model[])
{
	if(!is_valid_ent(entity))
		return FMRES_IGNORED;
	
	static szClassName[33]
	entity_get_string(entity, EV_SZ_classname, szClassName, charsmax(szClassName))
	
	if(!equal(szClassName, "weaponbox"))
		return FMRES_IGNORED;
	
	static iOwner
	
	iOwner = entity_get_edict(entity, EV_ENT_owner)
	
	if(equal(model, "models/w_mp5.mdl"))
	{
		static iStoredSVDID
		
		iStoredSVDID = find_ent_by_owner(ENG_NULLENT, "weapon_mp5navy", entity)
		
		if(!is_valid_ent(iStoredSVDID))
			return FMRES_IGNORED;
		
		if(g_has_k1ases[iOwner])
		{
			entity_set_int(iStoredSVDID, EV_INT_WEAPONKEY, k1ases_WEAPONKEY)
			g_has_k1ases[iOwner] = false
			g_delay[iOwner] = 0
			
			entity_set_model(entity, k1ases_W_MODEL)
			
			return FMRES_SUPERCEDE;
		}
	}
	
	
	return FMRES_IGNORED;
}

public give_k1ases(id)
{
	drop_weapons(id, 1);
	oldweap[id] = CSW_KNIFE
	new iWep2 = give_item(id,"weapon_mp5navy")
	if( iWep2 > 0 )
	{
		cs_set_weapon_ammo(iWep2, get_pcvar_num(cvar_clip_k1ases))
		cs_set_user_bpammo (id, CSW_MP5NAVY, get_pcvar_num(cvar_k1ases_ammo))
	}
	UTIL_PlayWeaponAnimation(id, 2)
	set_pdata_float(id, m_flNextAttack, 1.5, PLAYER_LINUX_XTRA_OFF)
	g_has_k1ases[id] = true;
	g_ammoclaw[id] = get_pcvar_num(cvar_k1asesammo )
	client_print(id,print_center,"[Claw Ammo: %d]",g_ammoclaw[id])
	
}

public fw_k1ases_AddToPlayer(k1ases, id)
{
	if(!is_valid_ent(k1ases) || !is_user_connected(id))
		return HAM_IGNORED;
	
	if(entity_get_int(k1ases, EV_INT_WEAPONKEY) == k1ases_WEAPONKEY)
	{
		g_has_k1ases[id] = true
		
		entity_set_int(k1ases, EV_INT_WEAPONKEY, 0)
		
		return HAM_HANDLED;
	}
	
	return HAM_IGNORED;
}

public fw_UseStationary_Post(entity, caller, activator, use_type)
{
	if (use_type == USE_STOPPED && is_user_connected(caller))
		replace_weapon_models(caller, get_user_weapon(caller))
}

public fw_Item_Deploy_Post(weapon_ent)
{
	static owner
	owner = fm_cs_get_weapon_ent_owner(weapon_ent)
	
	static weaponid
	weaponid = cs_get_weapon_id(weapon_ent)
	
	replace_weapon_models(owner, weaponid)
}

public CurrentWeapon(id)
{
	replace_weapon_models(id, read_data(2))
}

replace_weapon_models(id, weaponid) {
	switch (weaponid)
	{
		case CSW_MP5NAVY:
		{
			
			if(g_has_k1ases[id])
			{
				set_pev(id, pev_viewmodel2, k1ases_V_MODEL)
				set_pev(id, pev_weaponmodel2, k1ases_P_MODEL)
				if(oldweap[id] != CSW_MP5NAVY) 
				{
					UTIL_PlayWeaponAnimation(id, 2)
					set_pdata_float(id, m_flNextAttack, 1.5, PLAYER_LINUX_XTRA_OFF)
				}
				
			}
		}
	}
	oldweap[id] = weaponid
}

public fw_UpdateClientData_Post_k1asus(Player, SendWeapons, CD_Handle) {
	if(!is_user_alive(Player) || (get_user_weapon(Player) != CSW_MP5NAVY) || !g_has_k1ases[Player])
		return FMRES_IGNORED
	
	set_cd(CD_Handle, CD_flNextAttack, halflife_time () + 0.001)
	return FMRES_HANDLED
}

public fw_k1ases_PrimaryAttack(Weapon)
{
	new Player = get_pdata_cbase(Weapon, 41, 4)
	
	if (!g_has_k1ases[Player])
		return;
	
	pev(Player,pev_punchangle,cl_pushangle_k1asus[Player])
	
	g_clip_ammo[Player] = cs_get_weapon_ammo(Weapon)
}

public fwPlaybackEvent(flags, invoker, eventid, Float:delay, Float:origin[3], Float:angles[3], Float:fparam1, Float:fparam2, iParam1, iParam2, bParam1, bParam2)
{
	if ((eventid != g_orig_event_k1ases))
		return FMRES_IGNORED
	if (!(1 <= invoker <= g_MaxPlayers))
		return FMRES_IGNORED
	
	playback_event(flags | FEV_HOSTONLY, invoker, eventid, delay, origin, angles, fparam1, fparam2, iParam1, iParam2, bParam1, bParam2)
	return FMRES_SUPERCEDE
}

public fw_k1ases_PrimaryAttack_Post(Weapon)
{
	new Player = get_pdata_cbase(Weapon, 41, 4)
	
	new szClip, szAmmo
	get_user_weapon(Player, szClip, szAmmo)
	if(Player > 0 && Player < 33)
	{
		//if(!g_has_k1ases[Player])
		//{
		//if(szClip > 0) emit_sound(Player, CHAN_WEAPON, "weapons/famas-1.wav", VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
		//}
		if(g_has_k1ases[Player])
		{
			new Float:push[3]
			pev(Player,pev_punchangle,push)
			xs_vec_sub(push,cl_pushangle_k1asus[Player],push)
			
			xs_vec_mul_scalar(push,get_pcvar_float(cvar_recoil_k1ases),push)
			xs_vec_add(push,cl_pushangle_k1asus[Player],push)
			set_pev(Player,pev_punchangle,push)
			
			if (!g_clip_ammo[Player])
				return
			
			emit_sound(Player, CHAN_WEAPON, Fire_Sounds[0], VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
			UTIL_PlayWeaponAnimation(Player, 3)
			set_pdata_float(Player, m_flNextAttack, get_pcvar_float(cvar_k1ases_fire), PLAYER_LINUX_XTRA_OFF)
		}
	}
}

public fw_TakeDamage(victim, inflictor, attacker, Float:damage)
{
	if (victim != attacker && is_user_connected(attacker))
	{
		if(get_user_weapon(attacker) == CSW_MP5NAVY)
		{
			if(g_has_k1ases[attacker])
				SetHamParamFloat(4, damage * get_pcvar_float(cvar_dmg_k1ases))
		}
	}
}

public message_DeathMsg(msg_id, msg_dest, id)
{
	static szTruncatedWeapon[33], iAttacker, iVictim
	
	get_msg_arg_string(4, szTruncatedWeapon, charsmax(szTruncatedWeapon))
	
	iAttacker = get_msg_arg_int(1)
	iVictim = get_msg_arg_int(2)
	
	if(!is_user_connected(iAttacker) || iAttacker == iVictim)
		return PLUGIN_CONTINUE
	
	if(equal(szTruncatedWeapon, "famas") && get_user_weapon(iAttacker) == CSW_MP5NAVY)
	{
		if(g_has_k1ases[iAttacker])
			set_msg_arg_string(4, "famas")
	}
	
	return PLUGIN_CONTINUE
}

public k1ases__ItemPostFrame(weapon_entity) {
	new id = pev(weapon_entity, pev_owner)
	if (!is_user_connected(id))
		return HAM_IGNORED;
	
	if (!g_has_k1ases[id])
		return HAM_IGNORED;
	
	new Float:flNextAttack = get_pdata_float(id, m_flNextAttack, PLAYER_LINUX_XTRA_OFF)
	
	new iBpAmmo = cs_get_user_bpammo(id, CSW_MP5NAVY);
	new iClip = get_pdata_int(weapon_entity, m_iClip, WEAP_LINUX_XTRA_OFF)
	
	new fInReload = get_pdata_int(weapon_entity, m_fInReload, WEAP_LINUX_XTRA_OFF) 
	
	if( fInReload && flNextAttack <= 0.0 )
	{
		new j = min(get_pcvar_num(cvar_clip_k1ases) - iClip, iBpAmmo)
		
		set_pdata_int(weapon_entity, m_iClip, iClip + j, WEAP_LINUX_XTRA_OFF)
		cs_set_user_bpammo(id, CSW_MP5NAVY, iBpAmmo-j);
		
		set_pdata_int(weapon_entity, m_fInReload, 0, WEAP_LINUX_XTRA_OFF)
		fInReload = 0
	}
	
	return HAM_IGNORED;
}

public k1ases__Reload(weapon_entity) {
	new id = pev(weapon_entity, pev_owner)
	if (!is_user_connected(id))
		return HAM_IGNORED;
	
	if (!g_has_k1ases[id])
		return HAM_IGNORED;
	
	g_k1ases_TmpClip[id] = -1;
	
	new iBpAmmo = cs_get_user_bpammo(id, CSW_MP5NAVY);
	new iClip = get_pdata_int(weapon_entity, m_iClip, WEAP_LINUX_XTRA_OFF)
	
	if (iBpAmmo <= 0)
		return HAM_SUPERCEDE;
	
	if (iClip >= get_pcvar_num(cvar_clip_k1ases))
		return HAM_SUPERCEDE;
	
	
	g_k1ases_TmpClip[id] = iClip;
	
	return HAM_IGNORED;
}

public k1ases__Reload_Post(weapon_entity) {
	new id = pev(weapon_entity, pev_owner)
	if (!is_user_connected(id))
		return HAM_IGNORED;
	
	if (!g_has_k1ases[id])
		return HAM_IGNORED;
	
	if (g_k1ases_TmpClip[id] == -1)
		return HAM_IGNORED;
	
	set_pdata_int(weapon_entity, m_iClip, g_k1ases_TmpClip[id], WEAP_LINUX_XTRA_OFF)
	
	set_pdata_float(weapon_entity, m_flTimeWeaponIdle, k1ases_RELOAD_TIME, WEAP_LINUX_XTRA_OFF)
	
	set_pdata_float(id, m_flNextAttack, k1ases_RELOAD_TIME, PLAYER_LINUX_XTRA_OFF)
	
	set_pdata_int(weapon_entity, m_fInReload, 1, WEAP_LINUX_XTRA_OFF)
	
	// relaod animation
	UTIL_PlayWeaponAnimation(id, 1)
	
	return HAM_IGNORED;
}

public finish_reload(id)
{
	id -= TASK_RELOAD
	
	g_ammo[id] += g_reload_ammo[id]
	cs_set_user_bpammo(id, CSW_SALAMANDER, cs_get_user_bpammo(id, CSW_SALAMANDER) - g_reload_ammo[id])
	is_reloading[id] = false
}

public stop_fire(id)
{
	id -= TASK_FIRE
	
	is_firing[id] = false
	if(pev(id, pev_weaponanim) != SHOOT_END_ANIM)
		play_weapon_anim(id, SHOOT_END_ANIM)	
}

public throw_fire(id)
{
	new iEnt = create_entity("env_sprite")
	new Float:vfVelocity[3]
	
	velocity_by_aim(id, 500, vfVelocity)
	xs_vec_mul_scalar(vfVelocity, 0.4, vfVelocity)
	
	// add velocity of Owner for ent
	new Float:fOwnerVel[3], Float:vfAttack[3], Float:vfAngle[3]
	pev(id, pev_angles, vfAngle)
	//pev(id, pev_origin, vfAttack)
	get_weapon_attackment(id, vfAttack, 20.0)
	vfAttack[2] -= 7.0
	//vfAttack[1] += 7.0
	pev(id, pev_velocity, fOwnerVel)
	fOwnerVel[2] = 0.0
	xs_vec_add(vfVelocity, fOwnerVel, vfVelocity)
	
	// set info for ent
	set_pev(iEnt, pev_movetype, MOVETYPE_FLY)
	set_pev(iEnt, pev_rendermode, kRenderTransAdd)
	set_pev(iEnt, pev_renderamt, 150.0)
	set_pev(iEnt, PEV_ENT_TIME, get_gametime() + 1.5)	// time remove
	set_pev(iEnt, pev_scale, 0.2)
	set_pev(iEnt, pev_nextthink, halflife_time() + 0.05)
	
	set_pev(iEnt, pev_classname, fire_classname)
	engfunc(EngFunc_SetModel, iEnt, fire_spr_name)
	set_pev(iEnt, pev_mins, Float:{-1.0, -1.0, -1.0})
	set_pev(iEnt, pev_maxs, Float:{1.0, 1.0, 1.0})
	set_pev(iEnt, pev_origin, vfAttack)
	set_pev(iEnt, pev_gravity, 0.01)
	set_pev(iEnt, pev_velocity, vfVelocity)
	vfAngle[1] += 30.0
	set_pev(iEnt, pev_angles, vfAngle)
	set_pev(iEnt, pev_solid, SOLID_BBOX)
	set_pev(iEnt, pev_owner, id)
	set_pev(iEnt, pev_iuser2, 1)
}

public fw_think(iEnt)
{
	if ( !pev_valid(iEnt) ) return;
	
	new Float:fFrame, Float:fScale, Float:fNextThink
	pev(iEnt, pev_frame, fFrame)
	pev(iEnt, pev_scale, fScale)
	
	// effect exp
	new iMoveType = pev(iEnt, pev_movetype)
	if (iMoveType == MOVETYPE_NONE)
	{
		fNextThink = 0.015
		fFrame += 1.0
		
		if (fFrame > 21.0)
		{
			engfunc(EngFunc_RemoveEntity, iEnt)
			return
		}
	}
	
	// effect normal
	else
	{
		fNextThink = 0.045
		fFrame += 1.0
		fFrame = floatmin(21.0, fFrame)
	}
	
	fScale = (entity_range(iEnt, pev(iEnt, pev_owner)) / 500) * 3.0
	
	set_pev(iEnt, pev_frame, fFrame)
	set_pev(iEnt, pev_scale, fScale)
	set_pev(iEnt, pev_nextthink, halflife_time() + fNextThink)
	
	
	// time remove
	new Float:fTimeRemove
	pev(iEnt, PEV_ENT_TIME, fTimeRemove)
	if (get_gametime() >= fTimeRemove)
	{
		engfunc(EngFunc_RemoveEntity, iEnt)
		return;
	}
}

public fw_touch(ent, id)
{
	set_pev(ent, pev_movetype, MOVETYPE_NONE)
	set_pev(ent, pev_solid, SOLID_NOT)	
	
	if(!is_valid_ent(id))
		return FMRES_IGNORED
	
	if(!is_user_alive(id) || !is_user_connected(id))
		return FMRES_IGNORED
	
	if(pev(ent, pev_iuser2) == 1)
	{
		set_pev(ent, pev_iuser2, 0)
		
		static attacker, ent_kill
		
		attacker = pev(ent, pev_owner)
		ent_kill = fm_get_user_weapon_entity(id, CSW_KNIFE)
		
		
		ExecuteHam(Ham_TakeDamage, id, ent_kill, attacker, random_float(get_pcvar_float(cvar_dmgrd_start), get_pcvar_float(cvar_dmgrd_end)), DMG_BULLET)		
	}
	return FMRES_HANDLED
}

public Hook_Select(id)
{
	engclient_cmd(id, "weapon_p228")
	return PLUGIN_HANDLED
}


public dropcmd(id) {
	if(g_hasM79[id] && hasOnHandM79[id]) {
		new Float:Aim[3],Float:origin[3]
		VelocityByAim(id, 64, Aim)
		entity_get_vector(id,EV_VEC_origin,origin)
		
		origin[0] += Aim[0]
		origin[1] += Aim[1]
		
		new m79 = create_entity("info_target")
		entity_set_string(m79,EV_SZ_classname,"m79")
		entity_set_model(m79,m79_W_MODEL)	
		
		entity_set_size(m79,Float:{-2.0,-2.0,-2.0},Float:{5.0,5.0,5.0})
		entity_set_int(m79,EV_INT_solid,1)
		
		entity_set_int(m79,EV_INT_movetype,6)
		entity_set_int(m79, EV_INT_iuser1, grenade_count[id])
		entity_set_vector(m79,EV_VEC_origin,origin)
		g_hasM79[id] = false
		g_canShoot[id] = false
		grenade_count[id] = false
		hasOnHandM79[id] = false
		remowegun(id)
		remove_icon(id)
		set_task(0.15,"hud_clear",id)
		return PLUGIN_HANDLED
	} 
	return PLUGIN_CONTINUE
}

// remove gun  and save all guns
public remowegun(id) { 
	new wpnList[32] 
	new number
	get_user_weapons(id,wpnList,number) 
	for (new i = 0;i < number ;i++) { 
		if (wpnList[i] == CSW_P228) {
			fm_strip_user_gun(id, wpnList[i])
		}
	}
} 

//play anim
public playanim(player,anim)
{
	set_pev(player, pev_weaponanim, anim)
	message_begin(MSG_ONE, SVC_WEAPONANIM, {0, 0, 0}, player)
	write_byte(anim)
	write_byte(pev(player, pev_body))
	message_end()
}


// Current weapon player is holding
public Event_CurrentWeapon_m79(id)
{
	
	if(!is_user_connected(id))
		return
	
	// Read weapon ID
	new weaponID = read_data(2)
	
	if (weaponID == CSW_P228)
	{
		if (g_hasM79[id])
		{
			message_begin(MSG_ONE, get_user_msgid("CurWeapon"), {0,0,0}, id) 
			write_byte(1) 
			write_byte(CSW_KNIFE) 
			write_byte(0) 
			message_end()
			hasOnHandM79[id] = true
			remove_task(id+TASK_FRSTSHT)
			
			message_begin(MSG_ONE,get_user_msgid("StatusIcon"),{0,0,0},id);
			write_byte(1); // status (0=hide, 1=show, 2=flash)
			write_string("item_battery"); // sprite name
			write_byte(210) 
			write_byte(105)
			write_byte(30)
			message_end()
			
			set_task(0.1, "hud_init", id+TASK_HUDAMMO)
			
			if(!draw_wpn[id])
			{
				playanim(id, anim_draw)
				emit_sound(id, CHAN_WEAPON, m79_GRENADE_DRAW, VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
				// View model
				entity_set_string(id, EV_SZ_viewmodel, m79_V_MODEL)
				
				// Player model
				entity_set_string(id, EV_SZ_weaponmodel, m79_P_MODEL)
				canfire[id] = false
				draw_wpn[id] = true
				set_task(1.6,"delayshottsk", id+TASK_FRSTSHT)
			}
			else if(get_gametime() - g_last_shot_time[id] > delayshot) playanim(id, anim_idle)
			}
		else
		{
			// View model
			entity_set_string(id, EV_SZ_viewmodel, "models/v_p228.mdl")
			
			// Player model
			entity_set_string(id, EV_SZ_weaponmodel, "models/p_p228.mdl")
			hasOnHandM79[id] = false
			remove_icon(id)
			set_task(0.15,"hud_clear",id)
		}
	} 
	else 
	{
		draw_wpn[id] = false
		hasOnHandM79[id] = false
		remove_icon(id)
		set_task(0.15,"hud_clear",id)
	}
}

public fw_AddToPlayer_m79( iEnt, Player )
{
	if( pev_valid( iEnt ) && is_user_connected( Player ) )
	{
		if(g_hasM79[Player])
			Sprite(Player)
	}
} 


public remove_icon(id) 
{
	if(!is_user_connected(id))
		return
	
	message_begin(MSG_ONE,get_user_msgid("StatusIcon"),{0,0,0},id)
	write_byte(0) 
	write_string("item_battery") // sprite name
	message_end()
	return
}

public delayshottsk(taskid){
	canfire[ID_SHT] = true
}

// New round started
public Event_NewRound(id) {
	for (new i = 0; i < get_maxplayers(); i++) {
		g_hasM79[i] = false
	}
	m79 = 0
	draw_wpn[id] = false
	Event_CurrentWeapon_m79(id)
}

public Sprite(id) {
	
	message_begin(MSG_ONE, gmsgWeaponList, {0,0,0}, id)
	
	write_string("weapon_m79_sisa")
	
	write_byte(9)
	write_byte(52)
	write_byte(-1)
	write_byte(-1)
	write_byte(1)
	write_byte(3)
	write_byte(1)
	write_byte(0)
	message_end()
	
}



public event_start_m79()
{
	m79 = 0
	remove_m79()
}

public remove_m79() {
	new nextitem = find_ent_by_class(-1, "m79")
	while ((nextitem = find_ent_by_class(-1, "m79")) != 0) {
		remove_entity(nextitem)
	}
	return PLUGIN_CONTINUE
}		

// Player killed
public fw_PlayerKilled_m79(victim, attacker, shouldgib) {
	if (g_hasM79[victim])		
	{
		// Reset all
		g_hasM79[victim] = false
		g_canShoot[victim] = false
		g_FireM79[victim] = false
		grenade_count[victim] = 0
		
		// Remove HUD
		remove_task(victim+TASK_HUDAMMO)
		remowegun(victim)
		remove_icon(victim)
	}
}




//give wpn
public give_weapon(id, ammo){
	g_hasM79[id] = true
	g_canShoot[id] = true
	g_FireM79[id] = false
	give_item(id,"weapon_p228")
	if(ammo == -1) grenade_count[id] = 10
	else grenade_count[id] = ammo
	set_task(0.1, "hud_init", id+TASK_HUDAMMO)
}


// Command start
public fw_CmdStart_m79(id, uc_handle, seed)  {
	// Don't have our weapon	
	if (!hasOnHandM79[id] || !is_user_alive(id)){
		g_FireM79[id] = false
		return FMRES_IGNORED
	}
	// Get buttons	
	new buttons = get_uc(uc_handle, UC_Buttons)
	
	// Attack1 button pressed
	if(buttons & IN_ATTACK)
	{
		g_FireM79[id] = true
		
		// Remove attack button from their button mask
		buttons &= ~IN_ATTACK
		set_uc(uc_handle, UC_Buttons, buttons)
	}
	else 
		g_FireM79[id] = false	
	
	return FMRES_HANDLED
}

// Player think after
public fw_PlayerPostThink_m79(id)
{
	// Don't have our weapon	
	if (!g_hasM79[id])
		return FMRES_IGNORED
	
	// ignore dead
	if (!is_user_alive(id))
		return FMRES_IGNORED
	
	// Ignore zombies/nemesis	
	// If player is firing	
	if (g_FireM79[id] && hasOnHandM79[id])
	{
		// Grenades are more or equal to 1
		if (grenade_count[id] >= 1)
		{
			// Player can shoot
			if (get_gametime() - g_last_shot_time[id] > delayshot && canfire[id])
			{
				// Fire!!!
				FireGrenade(id)
				
				// Decrease nade count
				grenade_count[id]--
				
				// Without this HUD is not updating correctly
				set_task(0.1, "hud_init", id+TASK_HUDAMMO)
				
				// Remember last shot time
				g_last_shot_time[id] = get_gametime()
			}
		}
		else
		{
			// Don't have nades
			client_print(id, print_center, "Ð£ Ð²Ð°Ñ Ð±Ð¾Ð»ÑŒÑˆÐµ Ð½ÐµÑ‚ Ð³Ñ€Ð°Ð½Ð°Ñ‚!")
		}
	}
	
	return FMRES_IGNORED
}


//block anim standart wpn 
public fw_UpdateClientData_Post_m79(id, sendweapons, cd_handle) {
	// Don't have our weapon	
	if (!hasOnHandM79[id] || !is_user_alive(id)) return FMRES_IGNORED
	// Block default sounds	
	if (hasOnHandM79[id] && g_hasM79[id] && g_canShoot[id]) set_cd(cd_handle, CD_flNextAttack, halflife_time() + 0.001 ); 
	return FMRES_HANDLED
}

// Fire gremade
public FireGrenade(id) {
	// Set animation
	if(grenade_count[id]>1){ 
		playanim(id, anim_shot1)
		set_task(0.4,"reloadsnd",id)
		set_task(1.6,"reloadin",id)
		set_task(2.3,"reloadon",id)
		} else { 
		playanim(id, anim_shot2)
	}
	// Get origin. angle and velocity
	new Float:fAngle[3], Float:fVelocity[3]
	pev(id, pev_v_angle, fAngle)
	
	// Create ent
	new grenade = create_entity("info_target")
	
	// Not grenade
	if (!grenade) return PLUGIN_HANDLED
	
	// Classname
	entity_set_string(grenade, EV_SZ_classname, "m79_grenade")
	
	// Model
	entity_set_model(grenade, m79_GRENADE_MODEL)
	
	new Float:vOrigin[3],Float:vUp[3]
	pev(id, pev_origin, vOrigin)
	
	global_get(glb_v_up, vUp)
	new up = 17
	vOrigin[0] = vOrigin[0] + vUp[0] * up
	vOrigin[1] = vOrigin[1] + vUp[1] * up
	vOrigin[2] = vOrigin[2] + vUp[2] * up
	
	// Origin
	entity_set_origin(grenade, vOrigin)
	
	// Angles
	entity_set_vector(grenade, EV_VEC_angles, fAngle)
	
	// Size
	new Float:MinBox[3] = {-1.0, -1.0, -1.0}
	new Float:MaxBox[3] = {1.0, 1.0, 1.0}
	entity_set_vector(grenade, EV_VEC_mins, MinBox)
	entity_set_vector(grenade, EV_VEC_maxs, MaxBox)
	
	// Interaction
	entity_set_int(grenade, EV_INT_solid, SOLID_SLIDEBOX)
	
	// Movetype
	entity_set_int(grenade, EV_INT_movetype, MOVETYPE_TOSS)
	
	// Owner
	entity_set_edict(grenade, EV_ENT_owner, id)
	
	// Effects
	entity_set_int(grenade, EV_INT_effects, EF_BRIGHTLIGHT)
	
	// Velocity
	VelocityByAim(id, 1500, fVelocity)
	
	
	entity_set_vector(grenade, EV_VEC_velocity, fVelocity)
	
	// Launch sound
	emit_sound(grenade, CHAN_WEAPON, m79_GRENADE_SHOOT, VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
	
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
	write_byte(TE_BEAMFOLLOW) // Temporary entity ID
	write_short(grenade) // Entity
	write_short(sTrail) // Sprite index
	write_byte(3) // Life
	write_byte(1) // Line width
	write_byte(255) // Red
	write_byte(255) // Green
	write_byte(255) // Blue
	write_byte(255) // Alpha
	message_end() 
	return PLUGIN_CONTINUE
}	

public reloadsnd(id){
	emit_sound(id, CHAN_WEAPON, m79_GRENADE_CLIPOUT, VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
}
public reloadin(id){
	emit_sound(id, CHAN_WEAPON, m79_GRENADE_CLIPIN, VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
}
public reloadon(id){
	emit_sound(id, CHAN_WEAPON, m79_GRENADE_CLIPON, VOL_NORM, ATTN_NORM, 0, PITCH_NORM)
}

// We hit something!!!
public pfn_touch(pentru, ptd) {
	// If ent is valid
	if (pev_valid(pentru))
	{	
		// Get classnames
		static classname[32], classnameptd[32]
		pev(pentru, pev_classname, classname, 31)
		
		
		// Our ent
		if(equal(classname, "m79_grenade"))
		{
			// Get it's origin
			new Float:originF[3]
			pev(pentru, pev_origin, originF)
			
			// Draw explosion
			message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
			write_byte(TE_EXPLOSION) // Temporary entity ID
			engfunc(EngFunc_WriteCoord, originF[0]) // engfunc because float
			engfunc(EngFunc_WriteCoord, originF[1])
			engfunc(EngFunc_WriteCoord, originF[2])
			write_short(sExplo) // Sprite index
			write_byte(50) // Scale
			write_byte(15) // Framerate
			write_byte(0) // Flags
			message_end()
			
			// Draw smoke
			message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
			write_byte(TE_SMOKE) // Temporary entity IF
			engfunc(EngFunc_WriteCoord, originF[0]) // Pos X
			engfunc(EngFunc_WriteCoord, originF[1]) // Pos Y
			engfunc(EngFunc_WriteCoord, originF[2]) // Pos Z
			write_short(sSmoke) // Sprite index
			write_byte(75) // Scale
			write_byte(15) // Framerate
			message_end()
			
			// Get owner
			new owner = pev(pentru, pev_owner)
			
			new Max_Damage = get_pcvar_num(cvar_granade_max_damage)
			new Damage_Radius = get_pcvar_num(cvar_granade_damage_radius)
			
			// Loop through all players
			for(new i = 1; i < get_maxplayers(); i++)
			{
				// Alive...
				if (is_user_alive(i) == 1 && is_user_connected(owner))
				{
					
					// A zombie/nemesis
					if (get_user_team ( i ) == 1)
					{
						// Get victims origin and distance
						new VictimOrigin[3], Distance , origin[3]
						get_user_origin(i, VictimOrigin)
						// Get distance between victim and epicenter
						
						origin[0] = floatround(originF[0])
						origin[1] = floatround(originF[1])
						origin[2] = floatround(originF[2])
						
						Distance = get_distance(VictimOrigin, origin)
						
						if (Distance <= Damage_Radius)
						{
							// Start screen shake
							message_begin(MSG_ONE, g_msgScreenShake, {0,0,0}, i)
							write_short(1<<14) // Amount
							write_short(1<<14) // Duration
							write_short(1<<14) // Frequency
							message_end()
							new Damage
							Damage = Max_Damage - floatround(floatmul(float(Max_Damage), floatdiv(float(Distance), float(Damage_Radius))))
							make_knockback(i, originF, 1.5*float(Damage))	
							do_victim(i,owner,Damage)					
						}
					}
				}
				// Destroy ent
				set_pev(pentru, pev_flags, FL_KILLME)
			}
			// We hit breakable
			if(pev_valid(ptd)){
				pev(ptd, pev_classname, classnameptd, 31)
				if (equali(classnameptd, "func_breakable"))
				{
					// Destroy it
					force_use(pentru,ptd)
				}
		}	}
	}	
	if(is_valid_ent(pentru)) {
		new classname[32]
		entity_get_string(pentru,EV_SZ_classname,classname,31)
		
		if(equal(classname, "m79")) {
			if(is_valid_ent(ptd)) {
				new id = ptd
				if(id > 0 && id < 34) {
					
					// Pick up weapon
					give_weapon(id,entity_get_int(pentru, EV_INT_iuser1))
					remove_entity(pentru)
					
				}
			}
		}
	}
}	

public do_victim (victim,attacker,Damage) {
	
	new namek[32],namev[32],authida[35],authidv[35],teama[32],teamv[32]
	
	get_user_name(victim,namev,31)
	get_user_name(attacker,namek,31)
	get_user_authid(victim,authidv,34)
	get_user_authid(attacker,authida,34)
	get_user_team(victim,teamv,31)
	get_user_team(attacker,teama,31)
	static DamageTake[33]
	if(Damage >= get_user_health(victim)) {
		
		if(get_cvar_num("mp_logdetail") == 3) {
			
			log_message("^"%s<%d><%s><%s>^" attacked ^"%s<%d><%s><%s>^" with ^"grenade^" (hit ^"chest^") (Damage ^"%d^") (health ^"0^")",
			namek,get_user_userid(attacker),authida,teama,namev,get_user_userid(victim),authidv,teamv,Damage)
			
		}
		
		set_user_frags(attacker,get_user_frags(attacker) + 1 )
		
		set_msg_block(gmsgDeathMsg,BLOCK_ONCE)
		set_msg_block(gmsgScoreInfo,BLOCK_ONCE)
		
		ExecuteHamB(Ham_Killed, victim, attacker, 0)
		
		replace_dm(attacker,victim,0)
		
		log_message("^"%s<%d><%s><%s>^" killed ^"%s<%d><%s><%s>^" with ^"grenade^"",
		namek,get_user_userid(attacker),authida,teama,namev,get_user_userid(victim),authidv,teamv)
		
	}
	
	else {
		set_user_health(victim,get_user_health(victim) - Damage )
		
		if(get_cvar_num("mp_logdetail") == 3) {
			
			log_message("^"%s<%d><%s><%s>^" attacked ^"%s<%d><%s><%s>^" with ^"missile^" (hit ^"chest^") (Damage ^"%d^") (health ^"%d^")",
			namek,get_user_userid(attacker),authida,teama,namev,get_user_userid(victim),authidv,teamv,Damage,get_user_health(victim))
			
		}
		
	}
	if(DamageTake[attacker] >= 5000){
		DamageTake[attacker] -= 5000
	} else DamageTake[attacker] += Damage
	
}

public replace_dm (id,tid,tbody) {
	
	//Update killers scorboard with new info
	message_begin(MSG_ALL,gmsgScoreInfo)
	write_byte(id)
	write_short(get_user_frags(id))
	write_short(get_user_deaths(id))
	write_short(0)
	write_short(get_user_team(id))
	message_end()
	
	//Update victims scoreboard with correct info
	message_begin(MSG_ALL,gmsgScoreInfo)
	write_byte(tid)
	write_short(get_user_frags(tid))
	write_short(get_user_deaths(tid))
	write_short(0)
	write_short(get_user_team(tid))
	message_end()
	
	//Headshot Kill
	if (tbody == 1) {
		
		message_begin( MSG_ALL, gmsgDeathMsg,{0,0,0},0)
		write_byte(id)
		write_byte(tid)
		write_string("grenade")
		message_end()
		
	}
	
	//Normal Kill
	else {
		
		message_begin( MSG_ALL, gmsgDeathMsg,{0,0,0},0)
		write_byte(id)
		write_byte(tid)
		write_byte(0)
		write_string("grenade")
		message_end()
		
	}
	
	return PLUGIN_CONTINUE
	
}

// HUD init	
public hud_init(taskid) {
	new HudAmmo[65]
	
	format(HudAmmo, 64, "Grenades Left: [%d]", grenade_count[ID_HUDAMMO])
	
	message_begin(MSG_ONE, g_msgStatusText, {0,0,0}, ID_HUDAMMO)
	write_byte(0)
	write_string(HudAmmo) // Text
	message_end()
	
}
public hud_clear(id) {
	
	message_begin(MSG_ONE, g_msgStatusText, {0,0,0}, id)
	write_byte(0)
	write_string("") // Text
	message_end()
}

public FurienCurrentWeapon ( id ) {
	
	new szKnife = get_user_weapon ( id );
	if ( szKnife == CSW_KNIFE ) {
		if ( katana_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, katana_knife_v_model );
			set_pev ( id, pev_weaponmodel2, katana_knife_p_model );
			//set_task ( 0.1, "Katana_Damage", id );
			
		}
		
		if ( double_katana_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, double_katana_v_knife_model );
			set_pev ( id, pev_weaponmodel2, double_katana_p_knife_model );
			//set_task ( 0.1, "Double_Katana_Damage", id );
			
		}
		
		if ( super_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, super_knife_v_model );
			set_pev ( id, pev_weaponmodel2, super_knife_p_model );
			//set_task ( 0.1, "Super_Knife_Damage", id );
			
		}
		
		if ( infinity_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, infinity_knife_v_model );
			set_pev ( id, pev_weaponmodel2, infinity_knife_p_model );
			//set_task ( 0.1, "Infinity_Knife_Damage", id );
			
		}
		
		if ( vip_axe_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, axe_knife_v_model );
			set_pev ( id, pev_weaponmodel2, axe_knife_p_model );
			//set_task ( 0.1, "VIP_Axe_Knife_Damage", id );
			
		}
		
		if ( elf_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, elf_knife_model );
			//set_task ( 0.1, "Elf_Knife_Damage", id );
			
		}
		
		if ( ignes_knife [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, ignes_knife_model );
			//set_task ( 0.1, "Ignes_Knife_Damage", id );
			
		}
		
		if ( trainer [ id ] && szKnife == CSW_KNIFE ) {
			
			set_pev ( id, pev_viewmodel2, trainer_v_model );
			set_pev ( id, pev_weaponmodel2, trainer_p_model );
			//set_task ( 0.1, "Trainer_Knife_Damage", id );
			
		}
		
		if ( super_knife_shop [ id ] && szKnife == CSW_KNIFE ) {
			set_pev ( id, pev_viewmodel2, super_knife_shop_v_model );
			//set_task ( 0.1, "SK_Knife_Damage", id );
		}
		
		if ( super_knife_shop2 [ id ] && szKnife == CSW_KNIFE ) {
			set_pev ( id, pev_viewmodel2, super_knife_shop_v_model2 );
			set_pev ( id, pev_weaponmodel2, super_knife_shop_p_model2 );
			//set_task ( 0.1, "SK2_Knife_Damage", id );
		}
	}
	
	if ( strike_grenade [ id ] && szKnife == CSW_HEGRENADE && get_user_team ( id ) == 1 ) {
		
		set_pev ( id, pev_viewmodel2, strike_grenade_v_model );
		set_pev ( id, pev_weaponmodel2, strike_grenade_p_model );
		
	}
	
	if ( strike_grenade2 [ id ] && szKnife == CSW_HEGRENADE && get_user_team ( id ) == 1 ) {
		
		set_pev ( id, pev_viewmodel2, strike_grenade_v_model );
		set_pev ( id, pev_weaponmodel2, strike_grenade_p_model );
		
	}
	
	if ( strike_grenade3 [ id ] && szKnife == CSW_HEGRENADE && get_user_team ( id ) == 1 ) {
		
		set_pev ( id, pev_viewmodel2, strike_grenade_v_model );
		set_pev ( id, pev_weaponmodel2, strike_grenade_p_model );
		
	}
	
	else if ( !user_has_weapon ( id, CSW_KNIFE ) || ( !katana_knife [ id ] || !double_katana_knife || !super_knife || !infinity_knife || !vip_axe_knife || !ignes_knife || !elf_knife || !trainer ) )
	{
		give_item ( id, "weapon_knife" );
		return 1;
	}
	return 1;
}

public AntiFurienCurrentWeapon ( id ) {
	
	new szWeapon = get_user_weapon ( id );
	
	if ( szWeapon == CSW_MP5NAVY ) {
		if ( dual_mp5 [ id ] && szWeapon == CSW_MP5NAVY ) {
			
			set_pev ( id, pev_viewmodel2, dual_mp5_v_model );
			set_pev ( id, pev_weaponmodel2, dual_mp5_p_model );
			
		}
	}
	
	
	if ( szWeapon == CSW_P90 ) {
		if ( hunter [ id ] && szWeapon == CSW_P90 ) {
			
			set_pev ( id, pev_viewmodel2, hunter_v_model );
			set_pev ( id, pev_weaponmodel2, hunter_p_model );
			//set_task ( 0.1, "Hunter_Damage", id );
			
		}
		
		if ( thompson [ id ] && szWeapon == CSW_P90 ) {
			
			set_pev ( id, pev_viewmodel2, thompson_v_model );
			set_pev ( id, pev_weaponmodel2, thompson_p_model );
			//set_task ( 0.1, "Thompson_Damage", id );
			
		}
		
		if ( warrior [ id ] && szWeapon == CSW_P90 ) {
			
			set_pev ( id, pev_viewmodel2, warrior_v_model );
			set_pev ( id, pev_weaponmodel2, warrior_p_model );
			//set_task ( 0.1, "Warrior_Damage", id );
			
		}
		
		if ( deklowaz [ id ] && szWeapon == CSW_P90 ) {
			
			set_pev ( id, pev_viewmodel2, deklowaz_v_model );
			set_pev ( id, pev_weaponmodel2, deklowaz_p_model );
			//set_task ( 0.1, "Deklowaz_Damage", id );
			
		}
	}
	
	if ( szWeapon == CSW_GALIL ) {
		if ( mage [ id ] && szWeapon == CSW_GALIL ) {
			
			set_pev ( id, pev_viewmodel2, mage_v_model );
			set_pev ( id, pev_weaponmodel2, mage_p_model );
			//set_task ( 0.1, "Mage_Damage", id );
			
		}
	}
	
	if ( szWeapon == CSW_FAMAS ) {
		if ( rogue [ id ] && szWeapon == CSW_FAMAS ) {
			
			set_pev ( id, pev_viewmodel2, rogue_v_model );
			set_pev ( id, pev_weaponmodel2, rogue_p_model );
			//set_task ( 0.1, "Rogue_Damage", id );
			
		}
	}
	
	if ( szWeapon == CSW_SG552 ) {
		if ( shaman [ id ] && szWeapon == CSW_SG552 ) {
			
			set_pev ( id, pev_viewmodel2, shaman_v_model );
			set_pev ( id, pev_weaponmodel2, shaman_p_model );
			//set_task ( 0.1, "Shaman_Damage", id );
			
		}
	}
	
	if ( szWeapon == CSW_USP ) {
		if ( uspx [ id ] && szWeapon == CSW_USP ) {
			
			set_pev ( id, pev_viewmodel2, uspx_v_model );
			set_pev ( id, pev_weaponmodel2, uspx_p_model );
			
		}
	}
	
	if ( szWeapon == CSW_SMOKEGRENADE ) {
		if ( flare [ id ] && szWeapon == CSW_SMOKEGRENADE && get_user_team ( id ) == 2 ) {
			
			set_pev ( id, pev_viewmodel2, flare_v_model );
			
		}
	}
	
	return 1;
}

public Spawn(id) {
	remove_task(id);
	HasChose[id] = false;
	HE_Cooldown[id] = 0;
	GodMode_Cooldown[id] = 0;
	GodMode_DurationCooldown[id] = 0;
	Drop_Cooldown[id] = 0;
	Freeze_Cooldown[id] = 0;
	remove_freeze ( id );
	DragEnd ( id );
	Drag_Cooldown[id] = 0;
	Not_Cooldown[id] = false;
	Teleport_Cooldown[id] = 0;
}

public remove_freeze(id) {
	if (!Frozen[id] || !is_user_alive(id)) return;
	
	Frozen[id] = false;
	set_task(0.2, "set_normal", id);
	engfunc(EngFunc_EmitSound, id, CHAN_BODY, FROSTBREAK_SND[random_num(0, sizeof FROSTBREAK_SND - 1)], 1.0, ATTN_NORM, 0, PITCH_NORM);
	fm_set_rendering(id);
	static Float:origin2F[3];
	pev(id, pev_origin, origin2F);
	engfunc(EngFunc_MessageBegin, MSG_PVS, SVC_TEMPENTITY, origin2F, 0);
	write_byte(TE_BREAKMODEL);
	engfunc(EngFunc_WriteCoord, origin2F[0]);
	engfunc(EngFunc_WriteCoord, origin2F[1]);
	engfunc(EngFunc_WriteCoord, origin2F[2]+24.0);
	write_coord(16);
	write_coord(16);
	write_coord(16);
	write_coord(random_num(-50, 50));
	write_coord(random_num(-50, 50));
	write_coord(25);
	write_byte(10);
	write_short(FreezeSprite);
	write_byte(10);
	write_byte(25);
	write_byte(BREAK_GLASS);
	message_end();
}

public DragEnd(id) { // drags end function
	LastHook[id] = get_gametime();
	Hooked[id] = 0;
	BeamRemove(id);
	Drag_I[id] = false;
	Unable2move[id] = false;
	if(!Not_Cooldown[id] && HasPower[id] == 6) {
		Drag_Cooldown[id] = get_pcvar_num(CvarDragCooldown);
		set_task(1.0, "DragShowHUD", id, _, _, "b");
		Not_Cooldown[id] = true;
		set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
		if(get_pcvar_num(CvarDragCooldown) != 1) {
			show_hudmessage(id, "Puterea iti va reveni in: %d secunde",get_pcvar_num(CvarDragCooldown));
		}
		if(get_pcvar_num(CvarDragCooldown) == 1) {
			show_hudmessage(id, "Puterea iti va reveni in: %d secunda",get_pcvar_num(CvarDragCooldown));
		}
	}
}

public BeamRemove(id) { // remove beam
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY);
	write_byte(99);	//TE_KILLBEAM
	write_short(id);	//entity
	message_end();
}

//------| Client Death |------//
public Death() {
	new i = read_data ( 2 )
	remove_task(read_data(2));
	HE_Cooldown[read_data(2)] = 0;
	GodMode_Cooldown[read_data(2)] = 0;
	GodMode_DurationCooldown[read_data(2)] = 0;
	Drop_Cooldown[read_data(2)] = 0;
	Freeze_Cooldown[read_data(2)] = 0;
	Freeze_Cooldown[read_data(2)] = 0;
	remove_freeze ( i );
	BeamRemove ( i );
	Drag_Cooldown[read_data(2)] = 0;
	if (Hooked[read_data(2)])
		DragEnd ( i );
	
	Not_Cooldown[read_data(2)] = false;
	Teleport_Cooldown[read_data(2)] = 0;
}

//------| Client Power |------//
public Power(id)  {
	new target, body;
	static Float:start[3];
	static Float:aim[3];
	
	pev(id, pev_origin, start);
	fm_get_aim_origin(id, aim);
	
	start[2] += 16.0; // raise
	aim[2] += 16.0; // raise
	
	if ( is_user_alive(id) && HasPower[id] == 4) {
		
		if (Drop_Cooldown[id]) {
			ColorChat(id, GREEN, "%s Puterea iti va reveni in^4 %d^3 secunde .^4", szPrefix, Drop_Cooldown[id]);			
			return PLUGIN_CONTINUE;
		}
		get_user_aiming (id, target, body, CvarDropDistance);
		if(is_user_alive(target) && get_user_team(id) != get_user_team(target)) {
			message_begin(MSG_BROADCAST ,SVC_TEMPENTITY);
			write_byte(TE_EXPLOSION);
			engfunc(EngFunc_WriteCoord, aim[0]);
			engfunc(EngFunc_WriteCoord, aim[1]);
			engfunc(EngFunc_WriteCoord, aim[2]);
			write_short(DropSprite2);
			write_byte(10);
			write_byte(30);
			write_byte(4);
			message_end();
			
			emit_sound(id, CHAN_WEAPON, DROP_HIT_SND, VOL_NORM, ATTN_NORM, 0, PITCH_NORM);
			set_task ( 0.1, "Drop", target );
			message_begin(MSG_ONE, get_user_msgid("ScreenFade"), {0,0,0}, id);
			write_short(1<<10);
			write_short(1<<10);
			write_short(0x0000);
			write_byte(230);
			write_byte(0);
			write_byte(0);
			write_byte(50);
			message_end();
			message_begin(MSG_ONE, get_user_msgid("ScreenFade"), {0,0,0}, target);
			write_short(1<<10);
			write_short(1<<10);
			write_short(0x0000);
			write_byte(230);
			write_byte(0);
			write_byte(0);
			write_byte(50);
			message_end();
		}	
		message_begin(MSG_BROADCAST,SVC_TEMPENTITY);
		write_byte(0);
		engfunc(EngFunc_WriteCoord,start[0]);
		engfunc(EngFunc_WriteCoord,start[1]);
		engfunc(EngFunc_WriteCoord,start[2]);
		engfunc(EngFunc_WriteCoord,aim[0]);
		engfunc(EngFunc_WriteCoord,aim[1]);
		engfunc(EngFunc_WriteCoord,aim[2]);
		write_short(DropSprite); // sprite index
		write_byte(0); // start frame
		write_byte(30); // frame rate in 0.1's
		write_byte(20); // life in 0.1's
		write_byte(50); // line width in 0.1's
		write_byte(50); // noise amplititude in 0.01's
		write_byte(0); // red
		write_byte(100); // green
		write_byte(0); // blue
		write_byte(100); // brightness
		write_byte(50); // scroll speed in 0.1's
		message_end();
		Drop_Cooldown[id] = get_pcvar_num(CvarDropCooldown);
		set_task(1.0, "DropShowHUD", id, _, _, "b");
		set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
		if(get_pcvar_num(CvarDropCooldown) != 1) {
			show_hudmessage(id, "Puterea iti va reveni in: %d secunde",get_pcvar_num(CvarDropCooldown));
		}
		if(get_pcvar_num(CvarDropCooldown) == 1) {
			show_hudmessage(id, "Puterea iti va reveni in: %d secunda",get_pcvar_num(CvarDropCooldown));
		}
		return PLUGIN_HANDLED;
	}
	
	else if (is_user_alive(id) && HasPower[id] == 5) {
		if (Freeze_Cooldown[id]) {
			ColorChat(id, GREEN, "%s Puterea iti va reveni in^4 %d^3 secunde .^4", szPrefix,Freeze_Cooldown[id]);
			return PLUGIN_CONTINUE;
		}
		get_user_aiming (id, target, body, CvarFreezeDistance);
		if(is_user_alive(target) && get_user_team(id) != get_user_team(target)) {	
			set_task ( 0.1, "Freeze", target );
			
			message_begin(MSG_ONE, get_user_msgid("ScreenFade"), {0,0,0}, id);
			write_short(1<<10);
			write_short(1<<10);
			write_short(0x0000);
			write_byte(0);
			write_byte(100);
			write_byte(200);
			write_byte(50);
			message_end();
			message_begin(MSG_ONE, get_user_msgid("ScreenFade"), {0,0,0}, target);
			write_short(1<<10);
			write_short(1<<10);
			write_short(0x0000);
			write_byte(0);
			write_byte(100);
			write_byte(200);
			write_byte(50);
			message_end();
		}	
		message_begin(MSG_BROADCAST,SVC_TEMPENTITY);
		write_byte(0);
		engfunc(EngFunc_WriteCoord,start[0]);
		engfunc(EngFunc_WriteCoord,start[1]);
		engfunc(EngFunc_WriteCoord,start[2]);
		engfunc(EngFunc_WriteCoord,aim[0]);
		engfunc(EngFunc_WriteCoord,aim[1]);
		engfunc(EngFunc_WriteCoord,aim[2]);
		write_short(FreezeSprite3); // sprite index
		write_byte(0); // start frame
		write_byte(30); // frame rate in 0.1's
		write_byte(20); // life in 0.1's
		write_byte(50); // line width in 0.1's
		write_byte(50); // noise amplititude in 0.01's
		write_byte(0); // red
		write_byte(100); // green
		write_byte(200); // blue
		write_byte(100); // brightness
		write_byte(50); // scroll speed in 0.1's
		message_end();
		set_user_health ( target, get_user_health ( target ) - 5 );
		set_dhudmessage ( 255, 0, 0, 0.02, 0.90, 0, 6.0, 1.0 );
		show_dhudmessage ( id, "-5 HP" );
		Freeze_Cooldown[id] = get_pcvar_num(CvarFreezeCooldown);
		set_task(1.0, "FreezeShowHUD", id, _, _, "b");
		set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
		if(get_pcvar_num(CvarFreezeCooldown) != 1) {
			show_hudmessage(id, "Puterea iti va reveni in: %d secunde",get_pcvar_num(CvarFreezeCooldown));
		}
		if(get_pcvar_num(CvarFreezeCooldown) == 1) {
			show_hudmessage(id, "Puterea iti va reveni in: %d secunda",get_pcvar_num(CvarFreezeCooldown));
		}
		return PLUGIN_HANDLED;
	}
	else if  (is_user_alive(id) && HasPower[id] == 7) {	
		if (Teleport_Cooldown[id]) {
			ColorChat(id, GREEN, "%s Puterea iti va reveni in^4 %d^3 secunde .^4", szPrefix, Teleport_Cooldown[id]);
			return PLUGIN_CONTINUE;
		}
		if (teleport(id)) {
			emit_sound(id, CHAN_STATIC, SOUND_BLINK, 1.0, ATTN_NORM, 0, PITCH_NORM);
			remove_task(id);
			Teleport_Cooldown[id] = get_pcvar_num(CvarTeleportCooldown);
			set_task(1.0, "TeleportShowHUD", id, _, _, "b");
			set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
			if(get_pcvar_num(CvarTeleportCooldown) != 1) {
				show_hudmessage(id, "Puterea iti va reveni in: %d secunde",get_pcvar_num(CvarTeleportCooldown));
			}
			if(get_pcvar_num(CvarTeleportCooldown) == 1) {
				show_hudmessage(id, "Puterea iti va reveni in: %d secunda",get_pcvar_num(CvarTeleportCooldown));
			}
			return PLUGIN_HANDLED;
		}
		else {
			Teleport_Cooldown[id] = 0;
			ColorChat(id, GREEN, "%s Pozitia de teleportare este invalida .", szPrefix);
			return PLUGIN_HANDLED;
		}
		return PLUGIN_HANDLED;
	}
	return PLUGIN_CONTINUE;
}

bool:teleport(id) {
	new Float:vOrigin[3], Float:vNewOrigin[3],
	Float:vNormal[3], Float:vTraceDirection[3],
	Float:vTraceEnd[3];
	
	pev(id, pev_origin, vOrigin);
	
	velocity_by_aim(id, get_pcvar_num(CvarTeleportRange), vTraceDirection);
	xs_vec_add(vTraceDirection, vOrigin, vTraceEnd);
	
	engfunc(EngFunc_TraceLine, vOrigin, vTraceEnd, DONT_IGNORE_MONSTERS, id, 0);
	
	new Float:flFraction;
	get_tr2(0, TR_flFraction, flFraction);
	if (flFraction < 1.0) {
		get_tr2(0, TR_vecEndPos, vTraceEnd);
		get_tr2(0, TR_vecPlaneNormal, vNormal);
	}
	
	xs_vec_mul_scalar(vNormal, 40.0, vNormal); // do not decrease the 40.0
	xs_vec_add(vTraceEnd, vNormal, vNewOrigin);
	
	if (is_player_stuck(id, vNewOrigin))
		return false;
	
	emit_sound(id, CHAN_STATIC, SOUND_BLINK, 1.0, ATTN_NORM, 0, PITCH_NORM);
	tele_effect(vOrigin);
	
	engfunc(EngFunc_SetOrigin, id, vNewOrigin);
	
	tele_effect2(vNewOrigin);
	
	return true;
}

public cmdShowXp ( id ) {
	
	ColorChat ( id, GREEN, "%s Ai ^4%d^3 XP, iar levelul tau este ^4%s^3 .", szPrefix, eXP [ id ], Prefix [ Level [ id ] ] );
	ShowHud ( id );
	
}

public cmdSaveXp ( id ) {
	
	ColorChat ( id, GREEN, "%s Ti-ai salvat XP-ul cu succes .", szPrefix );
	ShowHud ( id );
	
}

public cmdShowLevel ( id ) {
	
	ColorChat ( id, GREEN, "%s Levelul tau este ^4%s^3 .", szPrefix, Prefix [ Level [ id ] ] );
	
}

public cmdShowLevels ( id ) {
	
	ColorChat ( id, GREEN, "%s In total sunt^4 30^3 levele .", szPrefix );
	
}

public cmdClearXp ( id ) {
	
	ColorChat ( id, GREEN, "%s Ti-ai sters XP-ul cu succes .", szPrefix );
	eXP [ id ] -= eXP [ id ];
	Level [ id ] -= Level [ id ];
}

public cmdRefreshXP ( id ) {
	
	Level [ id ] -= Level [ id ];
	ColorChat ( id, GREEN, "%s Ti-ai reimprospatat Xp-ul cu succes !", szPrefix );
	
	if ( !is_user_bot ( id ) && ( Level [ id ] < 30) && ( eXP [ id ] >= Levels [ Level [ id ] ] ) )
	{
		while ( eXP [ id ] >= Levels [ Level [ id ] ] )
		{
			Level [ id ] += 1;
		}
	}
}

public cmdRefreshXP2 ( id ) {
	
	Level [ id ] -= Level [ id ];
	// ColorChat ( id, GREEN, "%s Ti-ai reimprospatat Xp-ul cu succes !", szPrefix );
	
	if ( !is_user_bot ( id ) && ( Level [ id ] < 30) && ( eXP [ id ] >= Levels [ Level [ id ] ] ) )
	{
		while ( eXP [ id ] >= Levels [ Level [ id ] ] )
		{
			Level [ id ] += 1;
		}
	}
}

public cmdXpTop15 ( id ) {
	
	new i, count;
	static sort [ 33 ] [ 2 ], maxPlayers;
	
	if ( !maxPlayers ) maxPlayers = get_maxplayers ( );
	
	for ( i= 1; i <= maxPlayers; i++ )
	{
		sort [ count ][ 0 ] = i;
		sort [ count ][ 1 ] = Level [ i ];
		count++;
	}
	
	SortCustom2D ( sort,count, "stats_custom_compare" );
	
	new motd [ 1024 ], len;
	
	len = format ( motd, 1023, "<body bgcolor=#000000><center><font color=#FFB000><pre>" );
	len += format ( motd [ len ], 1023-len,"%s %-22.22s %3s^n", "#", "Name", "Level" );
	
	new players [ 32 ], num;
	get_players ( players, num );
	
	new b = clamp ( count,0,15 );
	
	new name [ 32 ], player;
	
	for ( new a = 0; a < b; a++ )
	{
		player = sort [ a ] [ 0 ];
		
		get_user_name ( player, name, 31 );		
		len += format ( motd [ len ], 1023-len,"%d %-22.22s %d^n", a+1, name, sort [ a ] [ 1 ] );
	}
	
	len += format ( motd [ len ], 1023-len,"</body></font></pre></center>" );
	show_motd(  id, motd, "Level Top 15" );
	
	return PLUGIN_CONTINUE;
}

public stats_custom_compare ( elem1 [ ], elem2 [ ] ) {
	
	if ( elem1 [ 1 ] > elem2 [ 1 ] ) return -1;
	else if ( elem1 [ 1 ] < elem2 [ 1 ] ) return 1;
		
	return 0;
}

public cmdGiveXp ( id, level, cid ) { 
	
	if(!cmd_access(id, level, cid, 3)) 
		return PLUGIN_HANDLED;
	
	new target[32], amount[21], reason[21], gplayers[32], players, num, i;
	
	read_argv(1, target, 31);
	read_argv(2, amount, 20);
	read_argv(3, reason, 20);
	
	new player = cmd_target(id, target, 8);
	
	if(!player)  
		return PLUGIN_HANDLED;
	
	new admin_name[32], player_name[32];
	get_user_name(id, admin_name, 31);
	get_user_name(player, player_name, 31);
	new expnum = str_to_num(amount);
	
	ColorChat ( 0, GREEN, "^4ADMIN ^3%s^1: ^1give ^4%s ^1xp to ^3%s ^1%s", admin_name, amount, player_name, reason );
	
	eXP [ player ] += expnum;
	cmdRefreshXP2 ( player );
	SaveData ( player );
	
	if(equali(target, "@All") || equali ( target, "all" ) ) {
		
		get_players(gplayers, num, "a");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			eXP [ players ] += expnum;
			SaveData(players);
			ColorChat ( 0, GREEN, "^4ADMIN ^3%s^1: ^1give ^4%s ^1xp to ^3All Players ^1%s", admin_name, amount, reason );
		}
	}
	
	return PLUGIN_CONTINUE;
}

public cmdSetXp ( id, level, cid ) { 
	
	if(!cmd_access(id, level, cid, 3)) 
		return PLUGIN_HANDLED;
	
	new target[32], amount[21], reason[21];
	
	read_argv(1, target, 31);
	read_argv(2, amount, 20);
	read_argv(3, reason, 20);
	
	new player = cmd_target(id, target, 8);
	
	if(!player)  
		return PLUGIN_HANDLED;
	
	new admin_name[32], player_name[32];
	get_user_name(id, admin_name, 31);
	get_user_name(player, player_name, 31);
	
	new expnum = str_to_num(amount);
	ColorChat ( 0, GREEN, "^4ADMIN ^3%s^1: ^1set ^4%s ^1xp to ^3%s ^1%s", admin_name, amount, player_name, reason );
	
	eXP [ player ] = expnum;
	cmdRefreshXP2 ( player );
	SaveData ( player );
	
	return PLUGIN_CONTINUE;
}

public fwd_setmodel(ent, const model[]) {
	if(!pev_valid(ent) || !equal(model[9], "smokegrenade.mdl"))
		return FMRES_IGNORED;
	
	static classname[32]; pev(ent, pev_classname, classname, 31);
	if(equal(classname, "grenade") && 1)
	{
		engfunc(EngFunc_SetModel, ent, flare_w_model);
		set_pev(ent, pev_effects, EF_BRIGHTLIGHT);
		set_pev(ent, pev_iuser4, 1337);
		set_pev(ent, pev_nextthink, get_gametime() + 999.9);
		fm_set_rendering2(ent, kRenderFxGlowShell, 150, 150, 250, kRenderNormal, 16);
		
		return FMRES_SUPERCEDE;
	}
	return FMRES_IGNORED;
}

public fwd_think(ent) if(pev_valid(ent) && is_ent_flare(ent))
engfunc(EngFunc_RemoveEntity, ent);

public GetRandomPlayer ( ) { 
	
	new players[32], count;    
	get_players(players, count, "a");    
	
	if(count)              
	{
		new iPlayer = players[random(count)];
		
		new Name[32];
		get_user_name(iPlayer, Name, charsmax(Name));
		set_user_credits ( iPlayer, get_user_credits ( iPlayer ) + 1 );
		
		ColorChat ( iPlayer, GREEN, "%s Felicitari ! Ai primit^4 1^3 credit .^4", szPrefix );
		ColorChat ( 0, GREEN, "%s^4 %s^3 a primit^4 1^3 credit .^4", szPrefix, Name );
	}
}  

public GiveQuadBarrel(id, itemid)
{
	if(itemid != g_quad_barrel)
		return PLUGIN_HANDLED
	
	g_had_qb[id] = 1
	new ent = give_item(id, "weapon_xm1014")
	
	cs_set_weapon_ammo(ent, get_pcvar_num(cvar_default_clip))
	cs_set_user_bpammo(id, CSW_QB, 10)
	
	set_pdata_float(id, 83, 1.0, 4)
	set_weapon_anim(id, 4)
	
	return PLUGIN_CONTINUE
}

public check_draw_weapon(id)
{
	set_task(0.001, "do_check", id)
}

public do_check(id)
{
	if(get_user_weapon(id) == CSW_QB && g_had_qb[id])
	{
		set_weapon_anim(id, 4)
	}
}

public event_curweapon_quad(id)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return
	if(get_user_weapon(id) != CSW_QB || !g_had_qb[id])
		return	
	
	set_pev(id, pev_viewmodel2, qb_v_model)
	set_pev(id, pev_weaponmodel2, qb_p_model)
	
	return 
}

public fw_UpdateClientData_Post_qb(id, sendweapons, cd_handle)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return FMRES_IGNORED	
	if(get_user_weapon(id) != CSW_QB || !g_had_qb[id])
		return FMRES_IGNORED
	
	set_cd(cd_handle, CD_flNextAttack, halflife_time() + 0.001) 
	
	return FMRES_HANDLED
}

public TraceAttack(iEnt, iAttacker, Float:flDamage, Float:fDir[3], pentru, iDamageType)
{
	if(!is_user_alive(iAttacker) || !is_user_connected(iAttacker))
		return HAM_IGNORED			
	if(get_user_weapon(iAttacker) != CSW_QB || !g_had_qb[iAttacker])
		return HAM_IGNORED
	
	static Float:flEnd[3]
	get_tr2(pentru, TR_vecEndPos, flEnd)
	
	make_bullet(iAttacker, flEnd)
	
	return HAM_HANDLED
}

public fw_takedmg(victim, inflictor, attacker, Float:damage, damagebits)
{
	if(!is_user_alive(victim) || !is_user_alive(attacker))
		return HAM_IGNORED
	
	
	if(get_user_weapon(attacker) == CSW_QB && g_had_qb[attacker])
	{
		static Float:random_start, Float:random_end
		
		random_start = get_pcvar_float(cvar_randmg_start)
		random_end = get_pcvar_float(cvar_randmg_end)
		
		SetHamParamFloat(4, random_float(random_start, random_end))
	}
	
	return HAM_HANDLED
}

public make_bullet(id, Float:Origin[3])
{
	// Find target
	new target, body
	get_user_aiming(id, target, body, 999999)
	
	if(target > 0 && target <= get_maxplayers())
	{
		new Float:fStart[3], Float:fEnd[3], Float:fRes[3], Float:fVel[3]
		pev(id, pev_origin, fStart)
		
		// Get ids view direction
		velocity_by_aim(id, 64, fVel)
		
		// Calculate position where blood should be displayed
		fStart[0] = Origin[0]
		fStart[1] = Origin[1]
		fStart[2] = Origin[2]
		fEnd[0] = fStart[0]+fVel[0]
		fEnd[1] = fStart[1]+fVel[1]
		fEnd[2] = fStart[2]+fVel[2]
		
		// Draw traceline from victims origin into ids view direction to find
		// the location on the wall to put some blood on there
		new res
		engfunc(EngFunc_TraceLine, fStart, fEnd, 0, target, res)
		get_tr2(res, TR_vecEndPos, fRes)
		
		// Show some blood :)
		message_begin(MSG_BROADCAST, SVC_TEMPENTITY) 
		write_byte(TE_BLOODSPRITE)
		write_coord(floatround(fStart[0])) 
		write_coord(floatround(fStart[1])) 
		write_coord(floatround(fStart[2])) 
		write_short(g_bloodspray)
		write_short(g_blood)
		write_byte(70)
		write_byte(random_num(1,2))
		message_end()
		
		
		} else {
		new decal = 41
		
		// Check if the wall hit is an entity
		if(target)
		{
			// Put decal on an entity
			message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
			write_byte(TE_DECAL)
			write_coord(floatround(Origin[0]))
			write_coord(floatround(Origin[1]))
			write_coord(floatround(Origin[2]))
			write_byte(decal)
			write_short(target)
			message_end()
			} else {
			// Put decal on "world" (a wall)
			message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
			write_byte(TE_WORLDDECAL)
			write_coord(floatround(Origin[0]))
			write_coord(floatround(Origin[1]))
			write_coord(floatround(Origin[2]))
			write_byte(decal)
			message_end()
		}
		
		// Show sparcles
		message_begin(MSG_BROADCAST, SVC_TEMPENTITY)
		write_byte(TE_GUNSHOTDECAL)
		write_coord(floatround(Origin[0]))
		write_coord(floatround(Origin[1]))
		write_coord(floatround(Origin[2]))
		write_short(id)
		write_byte(decal)
		message_end()
	}
}

public fm_cmdstart(id, uc_handle, seed)
{
	if(!is_user_alive(id) || !is_user_connected(id))
		return
	
	if(get_user_weapon(id) != CSW_QB || !g_had_qb[id])
		return 
	
	new CurButton
	CurButton = get_uc(uc_handle, UC_Buttons)
	
	if(CurButton & IN_RELOAD)
	{
		CurButton &= ~IN_RELOAD
		set_uc(uc_handle, UC_Buttons, CurButton)
		new ent = find_ent_by_owner(-1, "weapon_xm1014", id)
		
		if (!ent)
			return
		
		new fInReload = get_pdata_int(ent, m_fInReload, 4)
		
		new Float:flNextAttack ; flNextAttack = get_pdata_float(id, m_flNextAttack, 5)
		
		if (flNextAttack > 0.0)
			return
		
		if (fInReload)
		{
			set_weapon_anim(id, 0)
			return
		}
		if(cs_get_weapon_ammo(ent) >= get_pcvar_num(cvar_default_clip))
		{
			set_weapon_anim(id, 0)
			return
		}
		
		ham_reload(ent)
	}
	
	if(CurButton & IN_ATTACK2)
	{
		static Float:CurTime
		CurTime = get_gametime()
		
		if(CurTime - 4.0 > g_last_fire_qb[id])
		{
			static ent, ammo
			ent = find_ent_by_owner(-1, "weapon_xm1014", id)
			ammo = cs_get_weapon_ammo(ent)
			
			if(cs_get_weapon_ammo(ent) <= 0)
				return			
			
			for(new i = 0; i < ammo; i++)
			{
				ExecuteHamB(Ham_Weapon_PrimaryAttack, ent)
			}
			
			emit_sound(id, CHAN_WEAPON, qb_sound[4], 1.0, ATTN_NORM, 0, PITCH_NORM)
			set_weapon_anim(id, random_num(1, 2))
			
			g_last_fire_qb[id] = CurTime
		}
	}
	
	if(CurButton & IN_ATTACK)
	{
		static Float:CurTime
		CurTime = get_gametime()
		
		CurButton &= ~IN_ATTACK
		set_uc(uc_handle, UC_Buttons, CurButton)
		
		static ent
		ent = find_ent_by_owner(-1, "weapon_xm1014", id)
		
		if(cs_get_weapon_ammo(ent) <= 0 || get_pdata_int(ent, m_fInReload, XTRA_OFS_WEAPON))
			return
		
		if(CurTime - get_pcvar_float(cvar_delayattack) > g_last_fire2[id])
		{
			emit_sound(id, CHAN_WEAPON, qb_sound[4], 1.0, ATTN_NORM, 0, PITCH_NORM)
			
			ExecuteHamB(Ham_Weapon_PrimaryAttack, ent)
			set_weapon_anim(id, random_num(1, 2))
			
			g_last_fire2[id] = CurTime
		}
		
	}	
	
	return 
}

public ham_reload(iEnt)
{
	new id = pev(iEnt, pev_owner)
	
	if( g_had_qb[id])
	{
		static Cur_BpAmmo
		Cur_BpAmmo = cs_get_user_bpammo(id, CSW_QB)
		
		if(Cur_BpAmmo > 0)
		{
			set_pdata_int(iEnt, 55, 0, 4)
			set_pdata_float(id, 83, get_pcvar_float(cvar_reloadtime), 4)
			set_pdata_float(iEnt, 48, get_pcvar_float(cvar_reloadtime) + 0.5, 4)
			set_pdata_float(iEnt, 46, get_pcvar_float(cvar_reloadtime) + 0.25, 4)
			set_pdata_float(iEnt, 47, get_pcvar_float(cvar_reloadtime) + 0.25, 4)
			set_pdata_int(iEnt, 54, 1, 4)
			
			set_weapon_anim(id, 3)			
		}
		
		return HAM_HANDLED
	}
	return HAM_IGNORED
	
}

public fw_SetModel_qb(entity, model[])
{
	if(!is_valid_ent(entity))
		return FMRES_IGNORED;
	
	static szClassName[33]
	entity_get_string(entity, EV_SZ_classname, szClassName, charsmax(szClassName))
	
	if(!equal(szClassName, "weaponbox"))
		return FMRES_IGNORED;
	
	static iOwner
	iOwner = entity_get_edict(entity, EV_ENT_owner)
	
	if(equal(model, "models/w_xm1014.mdl"))
	{
		static weapon
		weapon = find_ent_by_owner(-1, "weapon_xm1014", entity)
		
		if(!is_valid_ent(weapon))
			return FMRES_IGNORED;
		
		if(g_had_qb[iOwner])
		{
			entity_set_int(weapon, EV_INT_impulse, 120)
			g_had_qb[iOwner] = 0
			set_pev(weapon, pev_iuser3, cs_get_weapon_ammo(weapon))
			entity_set_model(entity, qb_w_model)
			
			return FMRES_SUPERCEDE
		}
	}
	
	return FMRES_IGNORED;
}

public fw_item_addtoplayer_qb(ent, id)
{
	if(!is_valid_ent(ent))
		return HAM_IGNORED
	
	
	if(entity_get_int(ent, EV_INT_impulse) == 120)
	{
		g_had_qb[id] = 1
		cs_set_weapon_ammo(ent, pev(ent, pev_iuser3))
		
		entity_set_int(id, EV_INT_impulse, 0)
		check_draw_weapon(id)
		
		return HAM_HANDLED
	}		
	
	return HAM_HANDLED
}

public ham_priattack(ent)
{
	static id
	id = pev(ent, pev_owner)
	
	if(g_had_qb[id])
	{
		if(cs_get_weapon_ammo(ent) > 0)
		{
			emit_sound(id, CHAN_WEAPON, qb_sound[4], 1.0, ATTN_NORM, 0, PITCH_NORM)
		}
		
		set_pdata_float(id, 83, 0.3, 4)
	}
}

public ham_postframe(iEnt)
{
	new id = pev(iEnt, pev_owner)
	
	if(g_had_qb[id])
	{
		static iBpAmmo ; iBpAmmo = get_pdata_int(id, 381, XTRA_OFS_PLAYER)
		static iClip ; iClip = get_pdata_int(iEnt, m_iClip, XTRA_OFS_WEAPON)
		static iMaxClip ; iMaxClip = get_pcvar_num(cvar_default_clip)
		
		if(get_pdata_int(iEnt, m_fInReload, XTRA_OFS_WEAPON) && get_pdata_float(id, m_flNextAttack, 5) <= 0.0 )
		{
			new j = min(iMaxClip - iClip, iBpAmmo)
			set_pdata_int(iEnt, m_iClip, iClip + j, XTRA_OFS_WEAPON)
			set_pdata_int(id, 381, iBpAmmo-j, XTRA_OFS_PLAYER)
			
			set_pdata_int(iEnt, m_fInReload, 0, XTRA_OFS_WEAPON)
			cs_set_weapon_ammo(iEnt, get_pcvar_num(cvar_default_clip))
			
			return
		}
	}
}

public ShowSalamanderIcon ( id ) {
	
	if ( user_has_weapon ( id, CSW_M249 ) && ( salamander [ id ] ) ) {
		
		new iconstatus;
		iconstatus = get_user_msgid ( "StatusIcon" );
		
		if ( ! ( pev ( id,pev_button ) & FL_ONGROUND ) )
		{    
			message_begin ( MSG_ONE,iconstatus,{ 0,0,0 },id );
			write_byte ( 1 ); // status (0=hide, 1=show, 2=flash)
			write_string ( "dmg_heat" ); // sprite name
			write_byte ( 255 ); // red
			write_byte ( 0 ); // green
			write_byte ( 0 ); // blue
			message_end ( );
		}
		
	}
	
	if ( get_user_weapon ( id ) == CSW_M249 && ( salamander [ id ] ) ) {
		
		new iconstatus;
		iconstatus = get_user_msgid ( "StatusIcon" );
		
		if ( ! ( pev ( id,pev_button ) & FL_ONGROUND ) )
		{    
			message_begin ( MSG_ONE,iconstatus,{ 0,0,0 },id );
			write_byte ( 2 ); // status (0=hide, 1=show, 2=flash)
			write_string ( "dmg_heat" ); // sprite name
			write_byte ( 255 ); // red
			write_byte ( 0 ); // green
			write_byte ( 0 ); // blue
			message_end ( );
		}
	}
	
	if ( !salamander [ id ] || !user_has_weapon ( id, CSW_M249 ) ) {
		
		new iconstatus;
		iconstatus = get_user_msgid ( "StatusIcon" );
		
		if ( ! ( pev ( id,pev_button ) & FL_ONGROUND ) )
		{    
			message_begin ( MSG_ONE,iconstatus,{ 0,0,0 },id );
			write_byte ( 0 ); // status (0=hide, 1=show, 2=flash)
			write_string ( "dmg_heat" ); // sprite name
			write_byte ( 255 ); // red
			write_byte ( 0 ); // green
			write_byte ( 0 ); // blue
			message_end ( );
		}
	}
	
}

public fw_spawn ( id ) {
	if(g_had_salamander[id])
		g_had_salamander[id] = false
	
	if(task_exists(id+TASK_FIRE)) remove_task(id+TASK_FIRE)
	if(task_exists(id+TASK_RELOAD)) remove_task(id+TASK_RELOAD)
	
	remove_entity_name(fire_classname)
}

stock make_blood(const Float:vTraceEnd[3], Float:Damage, hitEnt) {
	new bloodColor = ExecuteHam(Ham_BloodColor, hitEnt);
	if(bloodColor == -1)
		return;
	
	new amount = floatround(Damage);
	
	amount *= 2; //according to HLSDK
	
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY);
	write_byte(TE_BLOODSPRITE);
	write_coord(floatround(vTraceEnd[0]));
	write_coord(floatround(vTraceEnd[1]));
	write_coord(floatround(vTraceEnd[2]));
	write_short(BloodSpray);
	write_short(BloodDrop);
	write_byte(bloodColor);
	write_byte(min(max(3, amount/10), 16));
	message_end();
}

// Make knockback
public make_knockback(Victim, Float:origin[3], Float:maxspeed) {
	// Get and set velocity
	new Float:fVelocity[3];
	kickback(Victim, origin, maxspeed, fVelocity);
	entity_set_vector(Victim, EV_VEC_velocity, fVelocity);
	
	return(1);
}

// Extra calulation for knockback
stock kickback(ent, Float:fOrigin[3], Float:fSpeed, Float:fVelocity[3]) {
	// Find origin
	new Float:fEntOrigin[3];
	entity_get_vector(ent, EV_VEC_origin, fEntOrigin);
	
	// Do some calculations
	new Float:fDistance[3];
	fDistance[0] = fEntOrigin[0] - fOrigin[0];
	fDistance[1] = fEntOrigin[1] - fOrigin[1];
	fDistance[2] = fEntOrigin[2] - fOrigin[2];
	new Float:fTime =(vector_distance(fEntOrigin,fOrigin) / fSpeed);
	fVelocity[0] = fDistance[0] / fTime;
	fVelocity[1] = fDistance[1] / fTime;
	fVelocity[2] = fDistance[2] / fTime;
	
	return(fVelocity[0] && fVelocity[1] && fVelocity[2]);
}

stock death_message(Killer, Victim, ScoreBoard, const Weapon[]) {
	// Block death msg
	set_msg_block(get_user_msgid("DeathMsg"), BLOCK_SET);
	ExecuteHamB(Ham_Killed, Victim, Killer, 2);
	set_msg_block(get_user_msgid("DeathMsg"), BLOCK_NOT);
	
	// Death
	make_deathmsg(Killer, Victim, 0, Weapon);
	cs_set_user_money(Killer, cs_get_user_money(Killer) + 300);
	
	// Update score board
	if(ScoreBoard) {
		message_begin(MSG_BROADCAST, get_user_msgid("ScoreInfo"));
		write_byte(Killer); // id
		write_short(pev(Killer, pev_frags)); // frags
		write_short(cs_get_user_deaths(Killer)); // deaths
		write_short(0); // class?
		write_short(get_user_team(Killer)); // team
		message_end();
		
		message_begin(MSG_BROADCAST, get_user_msgid("ScoreInfo"));
		write_byte(Victim); // id
		write_short(pev(Victim, pev_frags)); // frags
		write_short(cs_get_user_deaths(Victim)); // deaths
		write_short(0); // class?
		write_short(get_user_team(Victim)); // team
		message_end();
	}
}

stock get_damage_body(body, Float:damage) {
	switch(body) {
		case HIT_HEAD: damage *= 4.0;
			case HIT_STOMACH: damage *= 1.1;
			case HIT_CHEST: damage *= 1.5;
			case HIT_LEFTARM: damage *= 0.77;
			case HIT_RIGHTARM: damage *= 0.77;
			case HIT_LEFTLEG: damage *= 0.75;
			case HIT_RIGHTLEG: damage *= 0.75;
			default: damage *= 1.0;
	}
	
	return floatround(damage);
}	

stock fm_get_user_bpammo(index, weapon) {
	static offset
	switch(weapon) {
		case CSW_AWP: offset = OFFSET_AMMO_338MAGNUM
			case CSW_SCOUT, CSW_AK47, CSW_G3SG1: offset = OFFSET_AMMO_762NATO
			case CSW_M249: offset = OFFSET_AMMO_556NATOBOX
			case CSW_FAMAS, CSW_M4A1, CSW_AUG, 
			CSW_SG550, CSW_GALI, CSW_SG552: offset = OFFSET_AMMO_556NATO
		case CSW_M3, CSW_XM1014: offset = OFFSET_AMMO_BUCKSHOT
			case CSW_USP, CSW_UMP45, CSW_MAC10: offset = OFFSET_AMMO_45ACP
			case CSW_FIVESEVEN, CSW_P90: offset = OFFSET_AMMO_57MM
			case CSW_DEAGLE: offset = OFFSET_AMMO_50AE
			case CSW_P228: offset = OFFSET_AMMO_357SIG
			case CSW_GLOCK18, CSW_TMP, CSW_ELITE, 
			CSW_MP5NAVY: offset = OFFSET_AMMO_9MM
		default: offset = 0
	}
	return offset ? get_pdata_int(index, offset) : 0
}

stock fm_set_user_bpammo(index, weapon, amount) {
	static offset
	switch(weapon) {
		case CSW_AWP: offset = OFFSET_AMMO_338MAGNUM
			case CSW_SCOUT, CSW_AK47, CSW_G3SG1: offset = OFFSET_AMMO_762NATO
			case CSW_M249: offset = OFFSET_AMMO_556NATOBOX
			case CSW_FAMAS, CSW_M4A1, CSW_AUG, 
			CSW_SG550, CSW_GALI, CSW_SG552: offset = OFFSET_AMMO_556NATO
		case CSW_M3, CSW_XM1014: offset = OFFSET_AMMO_BUCKSHOT
			case CSW_USP, CSW_UMP45, CSW_MAC10: offset = OFFSET_AMMO_45ACP
			case CSW_FIVESEVEN, CSW_P90: offset = OFFSET_AMMO_57MM
			case CSW_DEAGLE: offset = OFFSET_AMMO_50AE
			case CSW_P228: offset = OFFSET_AMMO_357SIG
			case CSW_GLOCK18, CSW_TMP, CSW_ELITE, 
			CSW_MP5NAVY: offset = OFFSET_AMMO_9MM
		default: offset = 0
	}
	
	if(offset) 
		set_pdata_int(index, offset, amount)
	
	return 1
}

// Get Weapon Entity's CSW_ ID
stock fm_get_weapon_ent_id(ent) {
	return get_pdata_int(ent, OFFSET_WEAPONID, 4);
}

// Get Weapon Entity's Owner
stock fm_get_weapon_ent_owner(ent) {
	return get_pdata_cbase(ent, 41, 4);
}

// Drop all primary guns
stock drop_primary_weapons(Player) {
	// Get user weapons
	static weapons[32], num, i, weaponid;
	num = 0; // reset passed weapons count(bugfix)
	get_user_weapons(Player, weapons, num);
	
	// Loop through them and drop primaries
	for(i = 0; i < num; i++) {
		// Prevent re-indexing the array
		weaponid = weapons [i];
		
		// We definetely are holding primary gun
		if(((1<<weaponid) & PRIMARY_WEAPONS_BITSUM)) {
			// Get weapon entity
			static wname[32];
			get_weaponname(weaponid, wname, charsmax(wname));
			
			// Player drops the weapon and looses his bpammo
			engclient_cmd(Player, "drop", wname);
		}
	}
}

public RefreshWeapons ( id ) {
	
	strip_user_weapons ( id );
	give_item ( id, "weapon_knife" );
	remove_dragoncannon ( id );
	
	g_had_qb [ id ] = 0;
	dual_mp5 [ id ] = false;
	k1ases_weapon [ id ] = false;
	salamander [ id ] = false;
	SalamanderLimit [ id ] = false;
	katana_knife [ id ] = false;
	double_katana_knife [ id ] = false;
	super_knife [ id ] = false;
	infinity_knife [ id ] = false;
	elf_knife [ id ] = false;
	ignes_knife [ id ] = false;
	vip_axe_knife [ id ] = false;
	trainer [ id ] = false;
	thompson [ id ] = false;
	uspx [ id ] = false;
	hunter [ id ] = false;
	shaman [ id ] = false;
	mage [ id ] = false;
	rogue [ id ] = false;
	warrior [ id ] = false;
	druid [ id ] = false;
	deklowaz [ id ] = false;
	strike_grenade [ id ] = false;
	HasPower[id] = 0;
	Drop_Cooldown[id] = 0;
	UserHaveQuad [ id ] = false;
	UserHaveDragon [ id ] = false;
	g_hasM79[id] = false
	g_canShoot[id] = false
	g_last_shot_time[id] = 0.0
	grenade_count[id] = 0
	hasOnHandM79[id] = false
	remove_icon(id );
	UserHaveM79 [ id ] = false;
	
	HasSpeed[id] = false;
	HasTeleport[id] = false;
	
	UserHasChoosed [ id ] = false;
	UserHaveHeGrenade [ id ] = false;
	UserHaveGodMode [ id ] = false;
	UserHaveSuperKnife [ id ] = false;
	UserHaveNoClip [ id ] = false;
	UserHaveHpAndAp [ id ] = false;
	UserHaveDualMp5 [ id ] = false;
	
	if( get_user_flags(id) & VIP_ACCESS )
	{
		set_user_scoreattrib(id, 4);
	}
	
	VipBonus ( id );
	
	if ( get_user_team ( id ) == 2 ) {
		give_item ( id, "weapon_smokegrenade" );
		flare [ id ] = true;
	}
	
	ShowHud ( id );
	
	remove_task(id);
	HE_Cooldown[id] = 0;
	GodMode_Cooldown[id] = 0;
	GodMode_DurationCooldown[id] = 0;
	Drop_Cooldown[id] = 0;
	Freeze_Cooldown[id] = 0;
	Freeze_Cooldown[id] = 0;
	remove_freeze ( id );
	BeamRemove ( id );
	Drag_Cooldown[id] = 0;
	if (Hooked[id]) {
		DragEnd ( id );
	}
	
	Not_Cooldown[id] = false;
	Teleport_Cooldown[id] = 0;
}


public cmdCheckVIP ( id ) {
	
	if ( get_user_team ( id ) == 1 && is_user_alive ( id ) ) {
		
		if ( UserHasChoosed [ id ] ) {
			
			ColorChat ( id, GREEN, "%s Ti-ai ales runda aceasta puterea .", szPrefix );
			return 1;
		}
		
		else if ( !UserHasChoosed [ id ] ) {
			
			set_task ( 0.1, "cmdShowTVIPMenu", id );
			
		}
	}
	
	if ( get_user_team ( id ) == 2 && is_user_alive ( id ) ) {
		
		if ( UserHasChoosed [ id ] ) {
			
			ColorChat ( id, GREEN, "%s Ti-ai ales runda aceasta puterea .", szPrefix );
			return 1;
		}
		
		else if ( !UserHasChoosed [ id ] ) {
			
			set_task ( 0.1, "cmdShowCTVIPMenu", id );
			
		}
	}
	
	return 1
	
}

public cmdClassMenu ( id, level, cid ) {
	
	ShowHud ( id );
	
	if ( get_user_team ( id ) == 1 ) {
		
		
		new menu = menu_create ( "\rFurien Class \yMenu", "Class_Giver" );
		
		if ( Level [ id ] >= 0 || Level [ id ] >= 1 || Level [ id ] >= 2 || Level [ id ] >= 3 || Level [ id ] >= 4 ) { 
			menu_additem ( menu, "\yTrainer", "1", 0 );
		}
		
		else if ( Level [ id ] <= 0 || Level [ id ] <= 1 || Level [ id ] <= 2 || Level [ id ] <= 3 || Level [ id ] <= 4 ) {
			menu_additem ( menu, "\yTrainer \y[ \rLOCKED \y]", "1", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 5 || Level [ id ] >= 6 || Level [ id ] >= 7 || Level [ id ] >= 8 ) { 
			menu_additem ( menu, "\yAgnos", "2", 0 );
		}
		
		else if ( Level [ id ] <= 5 || Level [ id ] <= 6 || Level [ id ] <= 7 || Level [ id ] <= 8 ) {
			menu_additem ( menu, "\yAgnos \y[ \rLOCKED \y]", "2", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 9 || Level [ id ] >= 10 || Level [ id ] >= 11 || Level [ id ] >= 12 ) { 
			menu_additem ( menu, "\yXFother", "3", 0 );
		}
		
		else if ( Level [ id ] <= 9 || Level [ id ] <= 10 || Level [ id ] <= 11 || Level [ id ] <= 12 ) { 
			menu_additem ( menu, "\yXFother \y[ \rLOCKED \y]", "3", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 13 || Level [ id ] >= 14 || Level [ id ] >= 15 || Level [ id ] >= 16 ) { 
			menu_additem ( menu, "\ySamurai", "4", 0 );
		}
		
		else if ( Level [ id ] <= 13 || Level [ id ] <= 14 || Level [ id ] <= 15 || Level [ id ] <= 16 ) {
			menu_additem ( menu, "\ySamurai \y[ \rLOCKED \y]", "4", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 17 || Level [ id ] >= 18 || Level [ id ] >= 19 || Level [ id ] >= 20 ) { 
			menu_additem ( menu, "\yExtra Samurai", "5", 0 );
		}
		
		else if ( Level [ id ] <= 17 || Level [ id ] <= 18 || Level [ id ] <= 19 || Level [ id ] <= 20 ) { 
			menu_additem ( menu, "\yExtra Samurai \y[ \rLOCKED \y]", "5", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 21 || Level [ id ] >= 22 || Level [ id ] >= 23 || Level [ id ] >= 24 ) { 
			menu_additem ( menu, "\yIgnes", "6", 0 );
		}
		
		else if ( Level [ id ] <= 21 || Level [ id ] <= 22 || Level [ id ] <= 23 || Level [ id ] <= 24 ) {
			menu_additem ( menu, "\yIgnes \y[ \rLOCKED \y]", "6", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 25 || Level [ id ] >= 26 || Level [ id ] >= 27 || Level [ id ] >= 28 ) { 
			menu_additem ( menu, "\yElf", "7", 0 );
		}
		
		else if ( Level [ id ] <= 25 || Level [ id ] <= 26 || Level [ id ] <= 27 || Level [ id ] <= 28 ) { 
			menu_additem ( menu, "\yElf \y[ \rLOCKED \y]", "7", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 29 || Level [ id ] >= 30 ) { 
			menu_additem ( menu, "\yAlcadeias", "8", 0 );
		}
		
		else if ( Level [ id ] <= 29 || Level [ id ] <= 30 ) {
			menu_additem ( menu, "\yAlcadeias \y[ \rLOCKED \y]", "8", ADMIN_RCON );
		}
		
		menu_setprop ( menu, MPROP_EXIT, MEXIT_ALL );
		menu_display ( id, menu, 0 );
		
		
		return 1;
	}
	
	else if ( get_user_team ( id ) == 2 ) {
		
		
		new menu = menu_create ( "\rAntiFurien Class \yMenu", "Class_Giver" );
		
		if ( Level [ id ] >= 0 || Level [ id ] >= 1 || Level [ id ] >= 2 || Level [ id ] >= 3 || Level [ id ] >= 4 ) { 
			menu_additem ( menu, "\yDruid", "1", 0 );
		}
		
		else if ( Level [ id ] <= 0 || Level [ id ] <= 1 || Level [ id ] <= 2 || Level [ id ] <= 3 || Level [ id ] <= 4 ) { 
			menu_additem ( menu, "\yDruid \y[ \rLOCKED \y]", "1", ADMIN_RCON );
			
		}
		
		if ( Level [ id ] >= 5 || Level [ id ] >= 6 || Level [ id ] >= 7 || Level [ id ] >= 8 ) { 
			menu_additem ( menu, "\yHunter", "2", 0 );
		}
		
		else if ( Level [ id ] <= 5 || Level [ id ] <= 6 || Level [ id ] <= 7 || Level [ id ] <= 8 ) { 
			menu_additem ( menu, "\yHunter \y[ \rLOCKED \y]", "2", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 9 || Level [ id ] >= 10 || Level [ id ] >= 11 || Level [ id ] >= 12 ) { 
			menu_additem ( menu, "\yMage", "3", 0 );
		}
		
		else if ( Level [ id ] <= 9 || Level [ id ] <= 10 || Level [ id ] <= 11 || Level [ id ] <= 12 ) { 
			menu_additem ( menu, "\yMage \y[ \rLOCKED \y]", "3", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 13 || Level [ id ] >= 14 || Level [ id ] >= 15 || Level [ id ] >= 16 ) { 
			menu_additem ( menu, "\yRogue", "4", 0 );
		}
		
		else if ( Level [ id ] <= 13 || Level [ id ] <= 14 || Level [ id ] <= 15 || Level [ id ] <= 16 ) { 
			menu_additem ( menu, "\yRogue \y[ \rLOCKED \y]", "4", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 17 || Level [ id ] >= 18 || Level [ id ] >= 19 || Level [ id ] >= 20 ) { 
			menu_additem ( menu, "\yShaman", "5", 0 );
		}
		
		else if ( Level [ id ] <= 17 || Level [ id ] <= 18 || Level [ id ] <= 19 || Level [ id ] <= 20 ) { 
			menu_additem ( menu, "\yShaman \y[ \rLOCKED \y]", "5", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 21 || Level [ id ] >= 22 || Level [ id ] >= 23 || Level [ id ] >= 24 ) { 
			menu_additem ( menu, "\yWarlock", "6", 0 );
		}
		
		else if ( Level [ id ] <= 21 || Level [ id ] <= 22 || Level [ id ] <= 23 || Level [ id ] <= 24 ) { 
			menu_additem ( menu, "\yWarlock \y[ \rLOCKED \y]", "6", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 25 || Level [ id ] >= 26 || Level [ id ] >= 27 || Level [ id ] >= 28 ) { 
			menu_additem ( menu, "\yWarrior", "7", 0 );
		}
		
		else if ( Level [ id ] <= 25 || Level [ id ] <= 26 || Level [ id ] <= 27 || Level [ id ] <= 28 ) { 
			menu_additem ( menu, "\yWarrior \y[ \rLOCKED \y]", "7", ADMIN_RCON );
		}
		
		if ( Level [ id ] >= 29 || Level [ id ] >= 30 ) { 
			menu_additem ( menu, "\yDeklowaz", "8", 0 );
		}
		
		else if ( Level [ id ] <= 29 || Level [ id ] <= 30 ) { 
			menu_additem ( menu, "yDeklowaz \y[ \rLOCKED \y]", "8", ADMIN_RCON );
		}
		
		menu_setprop ( menu, MPROP_EXIT, MEXIT_ALL );
		menu_display ( id, menu, 0 );
		
		
		return 1;
	}
	
	return 1;
}

public Class_Giver ( id, menu, item ) {
	
	if( item == MENU_EXIT )
	{
		return 1;
	}
	
	new data [ 6 ], szName [ 64 ];
	new access, callback;
	menu_item_getinfo ( menu, item, access, data,charsmax ( data ), szName,charsmax ( szName ), callback );
	new key = str_to_num ( data );
	
	switch(key)
	{
		
		case 1:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Trainer^3 .", szPrefix );
					
					trainer [ id ] = true;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					druid [ id ] = false;
					strike_grenade [ id ] = false;
					strike_grenade2 [ id ] = false;
					strike_grenade3 [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.7 );
					give_item ( id, "weapon_flashbang" );
					//set_pev ( id, pev_viewmodel2, trainer_v_model );
					//set_pev ( id, pev_weaponmodel2, trainer_p_model );
					++g_Menu [ id ];
					++g_Menu [ id ];
				}
				
				
			}
			
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Druid^3 .", szPrefix );
					give_item ( id, "weapon_xm1014" );
					give_item ( id, "weapon_usp" );
					set_user_health ( id, 105 );
					set_user_armor ( id, 30 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_XM1014, 100 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					druid [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					
				}
			}
		}
		
		case 2:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Agnos^3 .", szPrefix );
					
					
					
					
					
					katana_knife [ id ] = false;
					trainer [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					
					infinity_knife [ id ] = true;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					strike_grenade [ id ] = false;
					strike_grenade2 [ id ] = false;
					strike_grenade3 [ id ] = false;
					druid [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.6 );
					set_user_health ( id, 120 );
					set_user_armor ( id, 60 );
					give_item ( id, "weapon_flashbang" );
					//set_pev ( id, pev_viewmodel2, infinity_knife_v_model );
					//set_pev ( id, pev_weaponmodel2, infinity_knife_p_model );
					++g_Menu [ id ];
					++g_Menu [ id ];
					
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Hunter^3 .", szPrefix );
					give_item ( id, "weapon_p90" );
					give_item ( id, "weapon_usp" );
					set_user_health ( id, 120 );
					set_user_armor ( id, 60 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_P90, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = true;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					druid [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					
				}
			}
		}
		
		case 3:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 XFother^3.^4", szPrefix );
					
					super_knife [ id ] = true;
					katana_knife [ id ] = false;
					trainer [ id ] = false;
					double_katana_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					elf_knife [ id ] = false;
					druid [ id ] = false;
					strike_grenade [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.6 );
					set_user_health ( id, 120 );
					set_user_armor ( id, 60 );
					give_item ( id, "weapon_hegrenade" );
					give_item ( id, "weapon_flashbang" );
					//set_pev ( id, pev_viewmodel2, super_knife_v_model );
					//set_pev ( id, pev_weaponmodel2, super_knife_p_model );
					++g_Menu [ id ];
					++g_Menu [ id ];
					
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Mage^3 .", szPrefix );
					give_item ( id, "weapon_galil" );
					give_item ( id, "weapon_usp" );
					set_user_gravity ( id, 0.7 );
					set_user_health ( id, 120 );
					set_user_armor ( id, 60 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_GALIL, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					druid [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = true;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					
				}
			}
		}
		
		case 4:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Samurai^3 + puterea de a arunca armele inamicului .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					
					
					
					
					
					katana_knife [ id ] = true;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					trainer [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					druid [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					strike_grenade2 [ id ] = true;
					deklowaz [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.6 );
					set_user_health ( id, 135 );
					set_user_armor ( id, 90 );
					give_item ( id, "weapon_hegrenade" );
					give_item ( id, "weapon_flashbang" );
					client_cmd(id, "bind v power");
					remove_task(id);
					if ( Drop_Cooldown [ id ] ) {
						set_task ( 0.1, "DropShowHUD", id );
					}
					HasPower [ id ] = 4;
					//set_pev ( id, pev_viewmodel2, katana_knife_v_model );
					//set_pev ( id, pev_weaponmodel2, katana_knife_p_model );
					++g_Menu [ id ];
					++g_Menu [ id ];
					
					
					
					
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Rogue^3 + puterea de a trage inamicul^4 [ drag ]^3 .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					give_item ( id, "weapon_famas" );
					give_item ( id, "weapon_usp" );
					set_user_gravity ( id, 0.7 );
					set_user_health ( id, 130 );
					set_user_armor ( id, 80 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_FAMAS, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = true;
					druid [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					client_cmd(id, "bind v +drag");
					remove_task(id);
					if(Drag_Cooldown[id]) {
						set_task ( 0.1, "DragShowHUD", id );
					}
					HasPower[id] = 6;
				}
			}
		}
		
		case 5:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Extra Samurai^3 + puterea de a arunca armele inamicului .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					
					
					double_katana_knife [ id ] = true;
					katana_knife [ id ] = false;
					trainer [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					druid [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					strike_grenade2 [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.5 );
					set_user_health ( id, 145 );
					set_user_armor ( id, 105 );
					give_item ( id, "weapon_hegrenade" );
					give_item ( id, "weapon_flashbang" );
					give_item ( id, "weapon_flashbang" );
					client_cmd(id, "bind v power");
					remove_task(id);
					if ( Drop_Cooldown [ id ] ) {
						set_task ( 0.1, "DropShowHUD", id );
					}
					HasPower [ id ] = 4;
					//set_pev ( id, pev_viewmodel2, double_katana_v_knife_model );
					//set_pev ( id, pev_weaponmodel2, double_katana_p_knife_model );
					++g_Menu [ id ];
					++g_Menu [ id ];
					
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Shaman^3 + puterea de a trage inamicul^4 [ drag ]^3 .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					give_item ( id, "weapon_sg552" );
					give_item ( id, "weapon_usp" );
					set_user_gravity ( id, 0.7 );
					set_user_health ( id, 145 );
					set_user_armor ( id, 90 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_SG552, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = true;
					mage [ id ] = false;
					rogue [ id ] = false;
					druid [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					client_cmd(id, "bind v +drag");
					remove_task(id);
					if(Drag_Cooldown[id]) {
						set_task ( 0.1, "DragShowHUD", id );
					}
					HasPower[id] = 6;
					
				}
			}
		}
		
		case 6:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Ignes^3 + puterea de a ingheta inamicul .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					
					
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					trainer [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					
					ignes_knife [ id ] = true;
					elf_knife [ id ] = false;
					vip_axe_knife [ id ] = false;
					druid [ id ] = false;
					strike_grenade2 [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.5 );
					set_user_health ( id, 185 );
					set_user_armor ( id, 150 );
					give_item ( id, "weapon_hegrenade" );
					give_item ( id, "weapon_flashbang" );
					give_item ( id, "weapon_flashbang" );
					client_cmd(id, "bind v power");
					remove_task(id);
					if(Freeze_Cooldown[id]) {
						set_task ( 0.1, "FreezeShowHUD", id );
					}
					HasPower[id] = 5;
					//set_pev ( id, pev_viewmodel2, ignes_knife_model );
					++g_Menu [ id ];
					++g_Menu [ id ];
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Warlock^3 + puterea de a trage perfect^4 [ norecoil ]^3 .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Puterea se activeaza automat .^4", szPrefix );
					give_item ( id, "weapon_p90" );
					give_item ( id, "weapon_usp" );
					set_user_gravity ( id, 0.6 );
					set_user_health ( id, 165 );
					set_user_armor ( id, 105 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_P90, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = true;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					druid [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					vip_axe_knife [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					remove_task(id);
					HasPower[id] = 8;
					
				}
			}
		}
		
		case 7:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Elf^3 + puterea de a ingheta inamicul .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					
					
					
					
					
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					trainer [ id ] = false;
					ignes_knife [ id ] = false;
					
					elf_knife [ id ] = true;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					druid [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					strike_grenade3 [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					set_user_gravity ( id, 0.4 );
					set_user_health ( id, 185 );
					set_user_armor ( id, 160 );
					give_item ( id, "weapon_hegrenade" );
					give_item ( id, "weapon_flashbang" );
					give_item ( id, "weapon_flashbang" );
					client_cmd(id, "bind v power");
					remove_task(id);
					if(Freeze_Cooldown[id]) {
						set_task ( 0.1, "FreezeShowHUD", id );
					}
					HasPower[id] = 5;
					//set_pev ( id, pev_viewmodel2, elf_knife_model );
					
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Warrior^3 + puterea de a trage perfect^4 [ norecoil ]^3 .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Puterea se activeaza automat .^4", szPrefix );
					give_item ( id, "weapon_p90" );
					give_item ( id, "weapon_usp" );
					set_user_gravity ( id, 0.6 );
					set_user_health ( id, 180 );
					set_user_armor ( id, 115 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_P90, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = false;
					druid [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = true;
					vip_axe_knife [ id ] = false;
					deklowaz [ id ] = false;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					remove_task(id);
					HasPower[id] = 8;
					
				}
			}
		}
		
		case 8:
		{
			if ( get_user_team ( id ) == 1 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Alcadeias^3 + puterea de a te teleporta .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					
					vip_axe_knife [ id ] = true;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					thompson [ id ] = false;
					uspx [ id ] = false;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					druid [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					trainer [ id ] = false;
					elf_knife [ id ] = false;
					strike_grenade3 [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					set_user_gravity ( id, 0.4 );
					set_user_health ( id, 200 );
					set_user_armor ( id, 200 );
					give_item ( id, "weapon_hegrenade" );
					give_item ( id, "weapon_flashbang" );
					give_item ( id, "weapon_flashbang" );
					++g_Menu [ id ];
					++g_Menu [ id ];
					client_cmd(id, "bind v power");
					remove_task(id);
					if(Teleport_Cooldown[id]) {
						set_task ( 0.1, "TeleportShowHUD", id );
					}
					HasPower[id] = 7;
					
				}
			}
			
			if ( get_user_team ( id ) == 2 ) {
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ti-ai ales odata clasa runda aceasta .", szPrefix );
				}
				
				else {
					ColorChat ( id, GREEN, "%s Clasa ta este acum^4 Deklowaz^3 + puterea de a te teleporta .^4", szPrefix );
					ColorChat ( id, GREEN, "%s Pentru activare apasa tasta^4 V^3 .^4", szPrefix );
					give_item ( id, "weapon_p90" );
					give_item ( id, "weapon_usp" );
					set_user_gravity ( id, 0.6 );
					set_user_health ( id, 200 );
					set_user_armor ( id, 130 );
					cs_set_user_bpammo ( id, CSW_USP, 100 );
					cs_set_user_bpammo ( id, CSW_P90, 200 );
					trainer [ id ] = false;
					katana_knife [ id ] = false;
					double_katana_knife [ id ] = false;
					super_knife [ id ] = false;
					infinity_knife [ id ] = false;
					ignes_knife [ id ] = false;
					elf_knife [ id ] = false;
					thompson [ id ] = false;
					druid [ id ] = false;
					uspx [ id ] = true;
					hunter [ id ] = false;
					shaman [ id ] = false;
					mage [ id ] = false;
					rogue [ id ] = false;
					warrior [ id ] = false;
					deklowaz [ id ] = true;
					super_knife_shop [ id ] = false;
					super_knife_shop2 [ id ] = false;
					++g_Menu [ id ];
					++g_Menu [ id ];
					client_cmd(id, "bind v power");
					remove_task(id);
					if(Teleport_Cooldown[id]) {
						set_task ( 0.1, "TeleportShowHUD", id );
					}
					HasPower[id] = 7;
					
				}
			}
		}
		
	}
	
	ShowHud ( id );
	if ( get_user_team ( id ) == 1 ) {
		cs_set_user_model(id, "furienxp")
	}
	
	if ( get_user_team ( id ) == 2 ) {
		cs_set_user_model(id, "furienxp2")
	}
	
	menu_destroy ( menu );
	return 1;
}

public cmdShop ( id, level, cid ) {
	
	if ( is_user_alive ( id ) ) {
		
		new menu = menu_create( "Shop Menu", "MenuShopGiver");
		
		if ( get_user_team ( id ) == 1 ) {
			menu_additem ( menu, "\ySuper Knife \y[ \r8000 $\y ]", "1", 0 );
		}
		
		menu_additem ( menu, "\yHE Grenade \y[ \r2500 $\y ]", "2", 0 );
		
		if ( get_user_team ( id ) == 2 ) {
			menu_additem ( menu, "\yDefuse Kit \y[ \r300 $\y ]", "3", 0 );
		}
		
		menu_additem ( menu, "\r+\y50 HP \y[ \r3000 $\y ]", "4", 0 );
		menu_additem ( menu, "\r+\y50 AP\r + \yHelmet \y[ \r2000 $\y ]", "5", 0 );
		
		
		menu_setprop(menu, MPROP_EXIT, MEXIT_ALL);
		menu_display(id, menu, 0);
		
		
		return 1;
	}
	
	else {
		ColorChat ( id, GREEN, "%s Nu poti folosi shop-ul cand esti mort.^4", szPrefix );
	}
	
	return 1;
}

public MenuShopGiver ( id, menu, item ) {
	
	if( item == MENU_EXIT )
	{
		return 1;
	}
	
	new data[6], szName[64];
	new access, callback;
	menu_item_getinfo(menu, item, access, data,charsmax(data), szName,charsmax(szName), callback);
	new key = str_to_num(data);
	
	switch(key)
	{
		case 1:
		{
			new iPret = cs_get_user_money ( id ) - 8000;
			if( iPret < 0 )
			{
				client_print( id, print_center, "Nu ai destui bani !" ); 
				return 1;
			}
			else
			{
				if ( Level [ id ] < 15 ) {
					super_knife_shop [ id ] = true;
					set_pev ( id, pev_viewmodel2, super_knife_shop_v_model );
					cs_set_user_money ( id, iPret );
				}
				
				else if ( Level [ id ] >= 15 ) {
					super_knife_shop2 [ id ] = true;
					set_pev ( id, pev_viewmodel2, super_knife_shop_v_model2 );
					set_pev ( id, pev_weaponmodel2, super_knife_shop_p_model2 );
					cs_set_user_money ( id, iPret );
				}
				
				
				return 1;
			}
			return 1
		}
		
		case 2:
		{
			new iPret = cs_get_user_money ( id ) - 2500;
			if( iPret < 0 )
			{
				client_print( id, print_center, "Nu ai destui bani !" );
				return 1;
			}
			else
			{
				give_item ( id, "weapon_hegrenade" );
				cs_set_user_money ( id, iPret );
				return 1;
			}
			return 1
		}
		
		case 3:
		{
			new iPret = cs_get_user_money ( id ) - 300;
			if( iPret < 0 )
			{
				client_print( id, print_center, "Nu ai destui bani !" );
				return 1;
			}
			else
			{
				give_item ( id, "item_thighpack" );
				cs_set_user_money ( id, iPret );
				return 1;
			}
			return 1
		}
		
		case 4:
		{
			new iPret = cs_get_user_money ( id ) - 3000;
			if( iPret < 0 )
			{
				client_print( id, print_center, "Nu ai destui bani !" );
				return 1;
			}
			else
			{
				set_dhudmessage ( 31, 201, 31, 0.02, 0.90, 0, 6.0, 1.0 );
				show_dhudmessage ( id, "+50 HP" );
				set_user_health ( id, get_user_health ( id ) + 50 );
				cs_set_user_money ( id, iPret );
				
				if ( get_user_team ( id ) == 2 && Level [ id ] < 20 ) {
					emit_sound(id, CHAN_ITEM, buy_AntiFurienHealth, 0.6, ATTN_NORM, 0, PITCH_NORM) 
					af ( id );
				}
				
				if ( get_user_team ( id ) == 2 && Level [ id ] >= 20 ) {
					emit_sound(id, CHAN_ITEM, buy_FurienHealth, 0.6, ATTN_NORM, 0, PITCH_NORM) 
					fr ( id );
				}
				
				return 1;
			}
			return 1
		}
		
		case 5:
		{
			new iPret = cs_get_user_money ( id ) - 2000;
			if( iPret < 0 )
			{
				client_print( id, print_center, "Nu ai destui bani !" );
				return 1;
			}
			else
			{
				set_dhudmessage ( 31, 201, 31, 0.20, 0.90, 0, 6.0, 1.0 );
				show_dhudmessage ( id, "+50 AP" );
				set_user_armor ( id, get_user_armor ( id ) + 50 );
				cs_set_user_money ( id, iPret );
				return 1;
			}
			return 1
		}
		
	}
	
	menu_destroy(menu);
	return 1
	
}

furien(origin[3]) { 
	message_begin(MSG_BROADCAST,SVC_TEMPENTITY) 
	write_byte(TE_SPRITE) 
	write_coord(origin[0]) 
	write_coord(origin[1]) 
	write_coord(origin[2]+=30) 
	write_short(g_FurienHealth) 
	write_byte(8) 
	write_byte(255) 
	message_end() 
} 

antifurien(origin[3]) { 
	message_begin(MSG_BROADCAST,SVC_TEMPENTITY) 
	write_byte(TE_SPRITE) 
	write_coord(origin[0]) 
	write_coord(origin[1]) 
	write_coord(origin[2]+=30) 
	write_short(g_AntiFurienHealth) 
	write_byte(8) 
	write_byte(255) 
	message_end() 
} 

public fr(victim) 
{ 
	new origin[3] 
	get_user_origin(victim,origin) 
	
	furien(origin) 
}

public af(victim) 
{ 
	new origin[3] 
	get_user_origin(victim,origin) 
	
	antifurien(origin) 
} 


public cmdShowTVIPMenu ( id, level, cid ) {
	
	new menu = menu_create ( "FurienVIP Menu", "MenuTVIPGiver" )
	
	if ( !UserHaveHpAndAp [ id ] ) {
		
		menu_additem ( menu, "\y225 HP\w &\y 225 AP", "1", VIP_ACCESS );
	}
	
	else if ( UserHaveHpAndAp [ id ] ) {
		
		menu_additem ( menu, "\w225 HP\y &\w 225 AP", "1", VIP_ACCESS );
	}
	
	if ( !UserHaveHeGrenade [ id ] ) {
		
		menu_additem ( menu, "\yHE GRENADE", "2", VIP_ACCESS );
	}
	
	else if ( UserHaveHeGrenade [ id ] ) {
		menu_additem ( menu, "\wHE GRENADE", "2", VIP_ACCESS );
	}
	
	if ( !UserHaveGodMode [ id ] ) {
		
		menu_additem ( menu, "\yGOD MODE", "3", VIP_ACCESS );
	}
	
	else if ( UserHaveGodMode [ id ] ) {
		
		menu_additem ( menu, "\wGOD MODE", "3", VIP_ACCESS );
	}
	
	if ( !UserHaveNoClip [ id ] ) {
		
		menu_additem ( menu, "\yNOCLIP", "4", VIP_ACCESS );
	}
	
	else if ( UserHaveNoClip [ id ] ) {
		
		menu_additem ( menu, "\wNOCLIP", "4", VIP_ACCESS );
	}
	
	if ( !UserHaveTeleport [ id ] ) {
		
		menu_additem ( menu, "\yTELEPORT", "5", VIP_ACCESS );
	}
	
	else if ( UserHaveTeleport [ id ] ) {
		
		menu_additem ( menu, "\wTELEPORT", "5", VIP_ACCESS );
	}
	
	if ( !UserHaveSuperKnife [ id ] ) {
		
		menu_additem ( menu, "\ySUPER KNIFE", "6", VIP_ACCESS );
	}
	
	else if ( UserHaveSuperKnife [ id ] ) {
		
		menu_additem ( menu, "\wSUPER KNIFE", "6", VIP_ACCESS );
	}
	
	menu_setprop(menu, MPROP_EXIT, MEXIT_ALL);
	menu_display(id, menu, 0);
	
	
	return 1
}

public cmdShowCTVIPMenu ( id, level, cid ) {
	
	new menu = menu_create ( "FurienVIP Menu", "MenuCTVIPGiver" )
	
	if ( !UserHaveHpAndAp [ id ] ) {
		
		menu_additem ( menu, "\y225 HP\w &\y 225 AP", "1", VIP_ACCESS );
	}
	
	else if ( UserHaveHpAndAp [ id ] ) {
		
		menu_additem ( menu, "\w225 HP\y &\w 225 AP", "1", VIP_ACCESS );
	}
	
	if ( !UserHaveHeGrenade [ id ] ) {
		
		menu_additem ( menu, "\yHE GRENADE", "2", VIP_ACCESS );
	}
	
	else if ( UserHaveHeGrenade [ id ] ) {
		menu_additem ( menu, "\wHE GRENADE", "2", VIP_ACCESS );
	}
	
	if ( !UserHaveGodMode [ id ] ) {
		
		menu_additem ( menu, "\yGOD MODE", "3", VIP_ACCESS );
	}
	
	else if ( UserHaveGodMode [ id ] ) {
		
		menu_additem ( menu, "\wGOD MODE", "3", VIP_ACCESS );
	}
	
	if ( !UserHaveNoClip [ id ] ) {
		
		menu_additem ( menu, "\yNOCLIP", "4", VIP_ACCESS );
	}
	
	else if ( UserHaveNoClip [ id ] ) {
		
		menu_additem ( menu, "\wNOCLIP", "4", VIP_ACCESS );
	}
	
	if ( !UserHaveTeleport [ id ] ) {
		
		menu_additem ( menu, "\yTELEPORT", "5", VIP_ACCESS );
	}
	
	else if ( UserHaveTeleport [ id ] ) {
		
		menu_additem ( menu, "\wTELEPORT", "5", VIP_ACCESS );
	}
	
	if ( !UserHaveSuperKnife [ id ] ) {
		
		menu_additem ( menu, "\yDUAL MP5", "6", VIP_ACCESS );
	}
	
	else if ( UserHaveSuperKnife [ id ] ) {
		
		menu_additem ( menu, "\wDUAL MP5", "6", VIP_ACCESS );
	}
	
	menu_setprop(menu, MPROP_EXIT, MEXIT_ALL);
	menu_display(id, menu, 0);
	
	
	return 1
}

public MenuCTVIPGiver ( id, menu, item )
{
	if( item == MENU_EXIT )
	{
		return 1
	}
	
	new data[6], szName[64];
	new access, callback;
	menu_item_getinfo(menu, item, access, data,charsmax(data), szName,charsmax(szName), callback);
	new key = str_to_num(data);
	
	switch(key)
	{
		case 1:
		{
			if ( UserHaveHpAndAp [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .^4", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveHpAndAp [ id ] ) {
				
				set_user_health ( id, 225 );
				set_user_armor ( id, 225 );
				ColorChat ( id, GREEN, "%s Ai primit^4 225 HP^3 &^4 225 AP^3 .^4", szPrefix );
				UserHaveHpAndAp [ id ] = true;
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 2:
		{
			if ( UserHaveHeGrenade [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveHeGrenade [ id ] ) {
				
				give_item ( id, "weapon_hegrenade" );
				ColorChat ( id, GREEN, "%s Vei primi o grenada^4 HE^3 odata la^4 15^3 secunde .^4", szPrefix );
				UserHaveHeGrenade [ id ] = true;
				set_task(15.0, "GiveMeAHeGrenade", id, _, _, "b");
				
				UserHaveGodMode [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 3:
		{
			if ( UserHaveGodMode [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveGodMode [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai primit^4 GodMode^3, apasa tasta^4 Z^3 pentru activare. ^4", szPrefix );
				UserHaveGodMode [ id ] = true;
				client_cmd ( id, "bind z vippower" );
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 4:
		{
			if ( UserHaveNoClip [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveNoClip [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai primit^4 NoClip^3, apasa tasta^4 Z^3 pentru activare. ^4", szPrefix );
				UserHaveNoClip [ id ] = true;
				client_cmd ( id, "bind z vippower" );
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 5:
		{
			if ( UserHaveTeleport [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveTeleport [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai primit puterea de a te^4 Teleporta^3, apasa tasta^4 Z^3 pentru activare. ^4", szPrefix );
				UserHaveTeleport [ id ] = true;
				client_cmd ( id, "bind z power" );
				remove_task(id);
				if(Teleport_Cooldown[id]) {
					set_task ( 0.1, "TeleportShowHUD", id );
				}
				
				HasPower[id] = 7;
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 6:
		{
			if ( UserHaveDualMp5 [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveDualMp5 [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai primit^4 Dual Mp5^3 . ^4", szPrefix );
				strip_user_weapons ( id );
				give_item ( id, "weapon_knife" );
				give_item ( id, "weapon_mp5navy" );
				give_item ( id, "weapon_hegrenade" );
				give_item ( id, "weapon_flashbang" );
				give_item ( id, "weapon_smokegrenade" );
				give_item ( id, "weapon_usp" );
				cs_set_user_bpammo ( id, CSW_MP5NAVY, 200 );
				cs_set_user_bpammo ( id, CSW_USP, 100 );
				dual_mp5 [ id ] = true;
				uspx [ id ] = true;
				flare [ id ] = true;
				
				UserHaveDualMp5 [ id ] = true;
				
				g_has_k1ases[id] = false
				g_delay[id] = 0
				g_ammoclaw[id] = 0
				UserHaveTeleport [ id ] = false;
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
	}
	
	menu_destroy(menu);
	return 1
	
}

public MenuTVIPGiver ( id, menu, item )
{
	if( item == MENU_EXIT )
	{
		return 1
	}
	
	new data[6], szName[64];
	new access, callback;
	menu_item_getinfo(menu, item, access, data,charsmax(data), szName,charsmax(szName), callback);
	new key = str_to_num(data);
	
	switch(key)
	{
		case 1:
		{
			if ( UserHaveHpAndAp [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .^4", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveHpAndAp [ id ] ) {
				
				set_user_health ( id, 225 );
				set_user_armor ( id, 225 );
				ColorChat ( id, GREEN, "%s Vei primi^4 225 HP^3 &^4 225 AP^3 in fiecare runda .^4", szPrefix );
				UserHaveHpAndAp [ id ] = true;
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 2:
		{
			if ( UserHaveHeGrenade [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveHeGrenade [ id ] ) {
				
				give_item ( id, "weapon_hegrenade" );
				ColorChat ( id, GREEN, "%s Vei primi o grenada^4 HE^3 odata la^4 15^3 secunde .^4", szPrefix );
				UserHaveHeGrenade [ id ] = true;
				set_task(15.0, "GiveMeAHeGrenade", id, _, _, "b");
				
				UserHaveGodMode [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 3:
		{
			if ( UserHaveGodMode [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveGodMode [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Vei primi^4 GodMode^3 in fiecare runda, apasa tasta^4 Z^3 pentru activare. ^4", szPrefix );
				UserHaveGodMode [ id ] = true;
				client_cmd ( id, "bind z vippower" );
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 4:
		{
			if ( UserHaveNoClip [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveNoClip [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Vei primi^4 NoClip^3 in fiecare runda, apasa tasta^4 Z^3 pentru activare. ^4", szPrefix );
				UserHaveNoClip [ id ] = true;
				client_cmd ( id, "bind z vippower" );
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveTeleport [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 5:
		{
			if ( UserHaveTeleport [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveTeleport [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Vei primi puterea de a te^4 Teleporta^3 in fiecare runda, apasa tasta^4 Z^3 pentru activare. ^4", szPrefix );
				UserHaveTeleport [ id ] = true;
				client_cmd ( id, "bind z power" );
				remove_task(id);
				if(Teleport_Cooldown[id]) {
					set_task ( 0.1, "TeleportShowHUD", id );
				}
				
				HasPower[id] = 7;
				
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
		case 6:
		{
			if ( UserHaveSuperKnife [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Ai deja aceasta putere .", szPrefix );
				return 1;
			}
			
			else if ( !UserHaveSuperKnife [ id ] ) {
				
				ColorChat ( id, GREEN, "%s Vei primi^4 SuperKnife^3 in fiecare runda. ^4", szPrefix );
				if ( Level [ id ] < 15 ) {
					super_knife_shop [ id ] = true;
					set_pev ( id, pev_viewmodel2, super_knife_shop_v_model );
				}
				
				else if ( Level [ id ] >= 15 ) {
					super_knife_shop2 [ id ] = true;
					set_pev ( id, pev_viewmodel2, super_knife_shop_v_model2 );
					set_pev ( id, pev_weaponmodel2, super_knife_shop_p_model2 );
				}
				
				UserHaveSuperKnife [ id ] = true;
				
				UserHaveTeleport [ id ] = false;
				UserHaveHeGrenade [ id ] = false;
				UserHaveGodMode [ id ] = false;
				UserHaveSuperKnife [ id ] = false;
				UserHaveNoClip [ id ] = false;
				UserHaveHpAndAp [ id ] = false;
				UserHasChoosed [ id ] = true;
			}
		}
		
	}
	
	menu_destroy(menu);
	return 1
	
}

public GiveMeAHeGrenade ( id ) {
	
	if ( UserHaveHeGrenade [ id ] ) {
		
		give_item ( id, "weapon_hegrenade" );
		set_hudmessage(0, 100, 225, 0.05, 0.60, 0, 6.0, 1.0)
		show_hudmessage ( id, "Ai primit o grenada HE" );
	}
}

public StopGodMode ( id ) {
	
	if ( UserHaveGodMode [ id ] ) {
		set_user_godmode ( id, 0 );
	}
}

public StopNoClip ( id ) {
	
	if ( UserHaveNoClip [ id ] ) {
		set_user_noclip ( id, 0 );
	}
	
}

public VIPpower ( id ) {
	
	if ( UserHaveGodMode [ id ] ) {
		
		set_user_godmode ( id, 1 );
		set_task(3.0, "StopGodMode", id );
		client_print ( id, print_center, "Ai GodMode penbtru 3 secunde" );
	}
	
	if ( UserHaveNoClip [ id ] ) {
		
		set_user_noclip ( id, 1 );
		set_task ( 4.0, "StopNoClip", id );
		client_print ( id, print_center, "Ai NoClip penbtru 4 secunde" );
	}
	
}

public VipBonus ( id ) {
	
	if ( UserHaveHpAndAp [ id ] ) {
		
		set_user_health ( id, 225 );
		set_user_armor ( id, 225 );
	}
	
	if ( UserHaveHeGrenade [ id ] ) {
		
		give_item ( id, "weapon_hegrenade" );
		set_task(15.0, "GiveMeAHeGrenade", id, _, _, "b");
	}
	
	if ( UserHaveGodMode [ id ] ) {
		
		client_cmd ( id, "bind z vippower" );
	}
	
	if ( UserHaveNoClip [ id ] ) {
		
		client_cmd ( id, "bind z vippower" );
		
	}
	
	if ( UserHaveTeleport [ id ] ) {
		
		client_cmd ( id, "bind z power" );
	}
	
	if ( UserHaveSuperKnife [ id ] ) {
		
		if ( Level [ id ] < 15 ) {
			super_knife_shop [ id ] = true;
			set_pev ( id, pev_viewmodel2, super_knife_shop_v_model );
		}
		
		else if ( Level [ id ] >= 15 ) {
			super_knife_shop2 [ id ] = true;
			set_pev ( id, pev_viewmodel2, super_knife_shop_v_model2 );
			set_pev ( id, pev_weaponmodel2, super_knife_shop_p_model2 );
		}
		
	}
	
}

public ForcePlayerSpeed ( id ) {
	
	if ( get_user_team ( id ) == 1 ) {
		
		if ( trainer [ id ] )
		{
			set_pev ( id, pev_maxspeed, 900.0 );
		} 
		
		if ( infinity_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 930.0 );
		} 
		
		if ( super_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 950.0 );
		} 
		
		if ( katana_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 1000.0 );
		} 
		
		if ( double_katana_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 1050.0 );
		} 
		
		if ( ignes_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 1100.0 );
		} 
		
		if ( elf_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 1150.0 );
		} 
		
		if ( vip_axe_knife [ id ] )
		{
			set_pev ( id, pev_maxspeed, 1200.0 );
		}
		
	}
}

public bomb_planted ( planter ) {
	
	eXP [ planter ] += 35;
	ColorChat ( planter, GREEN, "%s Ai primit^4 35^3 XP pentru plantarea bombei .^4", szPrefix );
	
	new originnn[3];
	get_user_origin ( planter, originnn, 0 );
	
	message_begin(MSG_PAS, SVC_TEMPENTITY, originnn);
	write_byte(TE_BEAMCYLINDER);
	write_coord(originnn[0]);
	write_coord(originnn[1]);
	write_coord(originnn[2]+10);
	write_coord(originnn[0]);
	write_coord(originnn[1]);
	write_coord(originnn[2]+60);
	write_short(TeleportSprite);
	write_byte(0);
	write_byte(0);
	write_byte(3);
	write_byte(60);
	write_byte(0);
	write_byte(255); //255
	write_byte(0); //255
	write_byte(0); //255
	write_byte(255); //255 //RED
	write_byte(0);
	message_end();
	
	UTIL_CreateBeamCylinder( originnn, 120, TeleportSprite, 0, 0, 6, 16, 0, 255, 0, 0, 255, 0 );
	UTIL_CreateBeamCylinder( originnn, 320, TeleportSprite, 0, 0, 6, 16, 0, 255, 51, 51, 255, 0 );
	UTIL_CreateBeamCylinder( originnn, 500, TeleportSprite, 0, 0, 6, 16, 0, 255, 102, 102, 255, 0 );
	
	new iPlayers[32]
	new iNum
	
	get_players( iPlayers, iNum, "e", "TERRORIST" )
	
	for( new i = 0; i < iNum; i++ )
	{
		g_CanUseHe[iPlayers[i]] = true;
	}
}

public bomb_defused ( defuser ) {
	eXP [ defuser ] += 35;
	ColorChat ( defuser, GREEN, "%s Ai primit^4 35^3 XP pentru defusarea bombei .^4", szPrefix );
	new originnn[3];
	get_user_origin ( defuser, originnn, 0 );
	message_begin(MSG_PAS, SVC_TEMPENTITY, originnn);
	write_byte(TE_BEAMCYLINDER);
	write_coord(originnn[0]);
	write_coord(originnn[1]);
	write_coord(originnn[2]+10);
	write_coord(originnn[0]);
	write_coord(originnn[1]);
	write_coord(originnn[2]+60);
	write_short(TeleportSprite);
	write_byte(0);
	write_byte(0);
	write_byte(3);
	write_byte(60);
	write_byte(0);
	write_byte(0); //255
	write_byte(0); //255
	write_byte(255); //255 //BLUE
	write_byte(255); //255 
	write_byte(0);
	message_end();
	
	Create_TE_SPRITETRAIL3( originnn, originnn, TeleportSprite3, 50, 10, 2, 50, 10 );
	
	/*---ScreenShake---*/
	message_begin(MSG_ONE , gMsgScreenShake , {0,0,0} ,defuser)
	write_short( 1<<14 );
	write_short( 1<<14 );
	write_short( 1<<14 );
	message_end();
}

public bomb_explode ( planter ) {
	eXP [ planter ] += 25;
	ColorChat ( planter, GREEN, "%s Ai primit^4 25^3 XP pentru explodarea bombei .^4", szPrefix );
}

public handle_say(id) {
	new said[192]
	read_args(said,192)
	if( ( containi(said, "who") != -1 && containi(said, "vips") != -1 ) || contain(said, "/vips") != -1 )
		set_task(0.1,"print_adminlist",id)
	return PLUGIN_CONTINUE
}

public print_adminlist(user) 
{
	new adminnames[33][32]
	new message[256]
	new contactinfo[256], contact[112]
	new id, count, x, len
	
	for(id = 1 ; id <= maxplayers ; id++)
		if(is_user_connected(id))
		if( get_user_flags(id) & VIP_ACCESS )
		get_user_name(id, adminnames[count++], 31)
	
	len = format(message, 255, "%s VIPS ONLINE: ",COLOR)
	if(count > 0) {
		for(x = 0 ; x < count ; x++) {
			len += format(message[len], 255-len, "%s%s ", adminnames[x], x < (count-1) ? ", ":"")
			if(len > 96 ) {
				print_message(user, message)
				len = format(message, 255, "%s ",COLOR)
			}
		}
		print_message(user, message)
	}
	else {
		len += format(message[len], 255-len, "Nici un VIP online.")
		print_message(user, message)
	}
	
	get_cvar_string("amx_contactinfo", contact, 63)
	if(contact[0])  {
		format(contactinfo, 111, "%s Cumpara VIP -- %s", COLOR, contact)
		print_message(user, contactinfo)
	}
}

print_message(id, msg[]) {
	message_begin(MSG_ONE, gmsgSayText, {0,0,0}, id)
	write_byte(id)
	write_string(msg)
	message_end()
}

public eDeath ( ) {
	
	new iKiller = read_data ( 1 );
	new iVictim = read_data ( 2 );
	new Headshot = read_data ( 3 );
	
	new weapon [ 32 ];
	read_data ( 4, weapon, sizeof ( weapon ) -1 );
	if ( iKiller == iVictim )
	{
		return 1;
	}
	new name [ 32 ];
	
	get_user_name ( iVictim, name, sizeof ( name ) -1 );
	
	if ( Headshot && get_user_team ( iKiller ) == 2 )
	{
		
		eXP [ iKiller ] += get_pcvar_num ( HsXp );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ HeadShot ]^3", szPrefix, get_pcvar_num ( HsXp ) + get_pcvar_num ( KillXp ) );
	}
	
	else if ( Headshot && get_user_team ( iKiller ) == 2 && get_user_flags ( iKiller ) & VIP_ACCESS )
	{
		
		eXP [ iKiller ] += get_pcvar_num ( HsXp ) + 10;
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ HeadShot ]^3", szPrefix, get_pcvar_num ( HsXp ) + get_pcvar_num ( KillXp ) );
	}
	
	else if ( Headshot && get_user_team ( iKiller ) == 1 && get_user_weapon ( iKiller ) == CSW_KNIFE && get_user_flags ( iKiller ) & VIP_ACCESS )
	{
		
		eXP [ iKiller ] += get_pcvar_num ( HsXp ) + 10;
		set_user_health ( iKiller, get_user_health ( iKiller ) + 35 );
		set_dhudmessage ( 31, 201, 31, 0.02, 0.90, 0, 6.0, 1.0 );
		show_dhudmessage ( iKiller, "+35 HP" );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP +^4 35^3 HP^4 [ HeadShot ]^3", szPrefix, get_pcvar_num ( HsXp ) + get_pcvar_num ( KillXp ) + 10 );
		
	}
	
	else if ( Headshot && get_user_team ( iKiller ) == 1 && get_user_weapon ( iKiller ) == CSW_KNIFE )
	{
		
		eXP [ iKiller ] += get_pcvar_num ( HsXp );
		set_user_health ( iKiller, get_user_health ( iKiller ) + 25 );
		set_dhudmessage ( 31, 201, 31, 0.02, 0.90, 0, 6.0, 1.0 );
		show_dhudmessage ( iKiller, "+25 HP" );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP +^4 25^3 HP^4 [ HeadShot ]^3", szPrefix, get_pcvar_num ( HsXp ) + get_pcvar_num ( KillXp ) );
		
	}
	
	else if ( equali ( weapon, "grenade" ) && Level [ iKiller ] == 0 || Level [ iKiller ] == 1 || Level [ iKiller ] == 2 || Level [ iKiller ] == 3 || Level [ iKiller ] == 4 || Level [ iKiller ] == 5 && get_user_flags ( iKiller ) & VIP_ACCESS )
	{
		eXP [ iKiller ] += get_pcvar_num ( HeXp ) + 5;
		set_user_credits ( iKiller, get_user_credits ( iKiller ) + 2 );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP +^4 2^3 credite^4 [ He Grenade ]^3", szPrefix, get_pcvar_num(HeXp) + get_pcvar_num(KillXp) + 5);
	}
	
	else if ( equali ( weapon, "grenade" ) && Level [ iKiller ] == 0 || Level [ iKiller ] == 1 || Level [ iKiller ] == 2 || Level [ iKiller ] == 3 || Level [ iKiller ] == 4 || Level [ iKiller ] == 5 )
	{
		eXP [ iKiller ] += get_pcvar_num ( HeXp );
		set_user_credits ( iKiller, get_user_credits ( iKiller ) + 1 );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP +^4 1^3 credit^4 [ He Grenade ]^3", szPrefix, get_pcvar_num(HeXp) + get_pcvar_num(KillXp) );
	}
	
	else if ( equali ( weapon, "grenade" ) && get_user_flags ( iKiller ) & VIP_ACCESS )
	{
		eXP [ iKiller ] += get_pcvar_num ( HeXp ) + 10;
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ He Grenade ]^3", szPrefix, get_pcvar_num(HeXp) + get_pcvar_num(KillXp) + 10 );
		
	}
	
	else if ( equali ( weapon, "grenade" ) )
	{
		eXP [ iKiller ] += get_pcvar_num ( HeXp );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ He Grenade ]^3", szPrefix, get_pcvar_num(HeXp) + get_pcvar_num(KillXp) );
		
	}
	
	else if ( equali ( weapon, "knife" ) && get_user_team ( iKiller ) == 2 && !salamander [ iKiller ] && get_user_flags ( iKiller ) & VIP_ACCESS )
	{
		eXP [ iKiller ] += get_pcvar_num ( KnifeXp ) + 10;
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ Knife ]^3", szPrefix, get_pcvar_num(KnifeXp) + get_pcvar_num(KillXp) + 10 );
		
	}
	
	else if ( equali ( weapon, "knife" ) && get_user_team ( iKiller ) == 2 && !salamander [ iKiller ] )
	{
		eXP [ iKiller ] += get_pcvar_num ( HeXp );
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ Knife ]^3", szPrefix, get_pcvar_num(KnifeXp) + get_pcvar_num(KillXp) );
		
	}
	
	else
	{
		ColorChat ( iKiller, GREEN, "%s Ai primit^4 %i^3 XP^4 [ Kill ]^3", szPrefix, get_pcvar_num ( KillXp ) );
		
	}
	
	if ( Level [ iVictim ] >= 25 ) {
		AddYellowBonusBox ( iVictim );
	}
	
	else if ( Level [ iVictim ] < 25 ) {
		AddBonusBox ( iVictim );
	}
	
	if ( get_user_team ( iKiller ) == 2 && get_user_team ( iVictim ) == 1 ) {
		cs_set_user_money ( iKiller, cs_get_user_money ( iKiller ) + 600 );
		eXP [ iKiller ] += get_pcvar_num ( KillXp );
	}
	
	if ( get_user_team ( iKiller ) == 1 && get_user_team ( iVictim ) == 2 ) {
		cs_set_user_money ( iKiller, cs_get_user_money ( iKiller ) + 500 );
		eXP [ iKiller ] += get_pcvar_num ( KillXp );
	}
	
	if ( Level [ iKiller ] <= 30 ) {
		ShowHud ( iKiller );
		return 1;
	}
	
	while ( eXP [ iKiller ] >= Levels [ Level [ iKiller ] ] ) {
		ColorChat ( iKiller, GREEN, "%s Felicitari ! Acum ai levelul ^4%s^3, cu ^4%i^3 XP.", szPrefix, Prefix [ Level [ iKiller ] ], eXP [ iKiller ] );
		Level [ iKiller ] ++;
		
	}
	
	SaveData ( iKiller );
	
	if ( !is_user_alive ( iVictim ) && get_user_team ( iVictim ) == 1 ) {
		new gMsgScreenFade = get_user_msgid ( "ScreenFade" );
		message_begin ( MSG_ONE_UNRELIABLE , gMsgScreenFade , {0,0,0} , iVictim );
		write_short ( (6<<10) ); // duration
		write_short ( (5<<10) ); // hold time
		write_short ( (1<<12) ); // fade type
		write_byte ( 255 );
		write_byte ( 0 );
		write_byte ( 0 );
		write_byte ( 170 );
		message_end ( );
	}
	
	else if ( !is_user_alive ( iVictim ) && get_user_team ( iVictim ) == 2 ) {
		new gMsgScreenFade = get_user_msgid ( "ScreenFade" );
		message_begin ( MSG_ONE_UNRELIABLE , gMsgScreenFade , {0,0,0} , iVictim );
		write_short ( (6<<10) ); // duration
		write_short ( (5<<10) ); // hold time
		write_short ( (1<<12) ); // fade type
		write_byte ( 0 );
		write_byte ( 0 );
		write_byte ( 255 );
		write_byte ( 170 );
		message_end ( );
	}
	
	return 1;
}

public CmdStart(id, uc_handle, seed) {
	new ent = fm_find_ent_by_class(id, ClassName)
	if(is_valid_ent(ent)) {
		new classname[32]	
		pev(ent, pev_classname, classname, 31)
		if (equal(classname, ClassName)) {
			
			if (pev(ent, pev_frame) >= 120)
				set_pev(ent, pev_frame, 0.0)
			else
				set_pev(ent, pev_frame, pev(ent, pev_frame) + 1.0)
			
			switch(pev(ent, pev_team))
			{
				case 1: 
				{ 	
				}	
				case 2: 
				{ 
				}
			}
		}
	}
}

public AddBonusBox(id) {
	
	if(is_user_connected(id) && cs_get_user_team(id) != CS_TEAM_SPECTATOR) {
		new ent = fm_create_entity("info_target")
		new origin[3]
		get_user_origin(id, origin, 0)
		set_pev(ent,pev_classname, ClassName)
		switch(cs_get_user_team(id))
		{
			case CS_TEAM_T: { 
				engfunc(EngFunc_SetModel,ent, Model[1])
				set_pev(ent,pev_team, 2)
			}
			
			case CS_TEAM_CT: {
				engfunc(EngFunc_SetModel,ent, Model[0])	
				set_pev(ent,pev_team, 1)
			}
		}
		set_pev(ent,pev_mins,Float:{-10.0,-10.0,0.0})
		set_pev(ent,pev_maxs,Float:{10.0,10.0,25.0})
		set_pev(ent,pev_size,Float:{-10.0,-10.0,0.0,10.0,10.0,25.0})
		engfunc(EngFunc_SetSize,ent,Float:{-10.0,-10.0,0.0},Float:{10.0,10.0,25.0})
		
		set_pev(ent,pev_solid,SOLID_BBOX)
		set_pev(ent,pev_movetype,MOVETYPE_TOSS)
		
		new Float:fOrigin[3]
		IVecFVec(origin, fOrigin)
		set_pev(ent, pev_origin, fOrigin)
	}
}

public AddYellowBonusBox(id)
{
	if(is_user_connected(id) && cs_get_user_team(id) != CS_TEAM_SPECTATOR) {
		new ent = fm_create_entity("info_target")
		new origin[3]
		get_user_origin(id, origin, 0)
		set_pev(ent,pev_classname, ClassName)
		switch(cs_get_user_team(id))
		{
			case CS_TEAM_T: { 
				engfunc(EngFunc_SetModel,ent, Model_Yellow[1])
				set_pev(ent,pev_team, 2)
			}
			
			case CS_TEAM_CT: {
				engfunc(EngFunc_SetModel,ent, Model_Yellow[0])	
				set_pev(ent,pev_team, 1)
			}
		}
		set_pev(ent,pev_mins,Float:{-10.0,-10.0,0.0})
		set_pev(ent,pev_maxs,Float:{10.0,10.0,25.0})
		set_pev(ent,pev_size,Float:{-10.0,-10.0,0.0,10.0,10.0,25.0})
		engfunc(EngFunc_SetSize,ent,Float:{-10.0,-10.0,0.0},Float:{10.0,10.0,25.0})
		
		set_pev(ent,pev_solid,SOLID_BBOX)
		set_pev(ent,pev_movetype,MOVETYPE_TOSS)
		
		new Float:fOrigin[3]
		IVecFVec(origin, fOrigin)
		set_pev(ent, pev_origin, fOrigin)
	}
}

public Touch(toucher, touched)
{
	if (!is_user_alive(toucher) || !pev_valid(touched))
		return FMRES_IGNORED
	
	new classname[32]	
	pev(touched, pev_classname, classname, 31)
	if (!equal(classname, ClassName))
		return FMRES_IGNORED
	
	if(get_user_team(toucher) == pev(touched, pev_team) )
	{
		GiveBonusBox(toucher)
		set_pev(touched, pev_effects, EF_NODRAW)
		set_pev(touched, pev_solid, SOLID_NOT)
		remove_entity(touched);
	}
	
	return FMRES_IGNORED
}

public Touch_Yellow ( toucher, touched ) {
	
	if (!is_user_alive(toucher) || !pev_valid(touched))
		return FMRES_IGNORED
	
	new classname[32]	
	pev(touched, pev_classname, classname, 31)
	if (!equal(classname, ClassName))
		return FMRES_IGNORED
	
	if(get_user_team(toucher) == pev(touched, pev_team) )
	{
		GiveYellowBonusBox(toucher)
		set_pev(touched, pev_effects, EF_NODRAW)
		set_pev(touched, pev_solid, SOLID_NOT)
		remove_entity(touched);
	}
	
	return FMRES_IGNORED
}

public event_cur_weapon(id) {
	if(HasSpeed[id] && cs_get_user_team(id) == CS_TEAM_T && get_user_maxspeed(id) < get_pcvar_float(CvarFurienSpeed)) {
		set_user_maxspeed(id, get_pcvar_float(CvarFurienSpeed));
	}
	if(HasSpeed[id] && cs_get_user_team(id) == CS_TEAM_CT && get_user_maxspeed(id) < get_pcvar_float(CvarAntiFurienSpeed)) {
		set_user_maxspeed(id, get_pcvar_float(CvarAntiFurienSpeed));
	}
	
	if ( LowSpeed [ id ] && cs_get_user_team ( id ) == CS_TEAM_T ) {
		set_user_maxspeed ( id, get_user_maxspeed ( id ) - 250 );
	}
	
	if ( LowSpeed [ id ] && cs_get_user_team ( id ) == CS_TEAM_CT ) {
		set_user_maxspeed ( id, get_user_maxspeed ( id ) - 70 );
	}
}

public wrongeffect ( id ) {
	new gMsgScreenFade = get_user_msgid ( "ScreenFade" );
	message_begin ( MSG_ONE_UNRELIABLE , gMsgScreenFade , {0,0,0} , id );
	write_short ( (6<<10) ); // duration
	write_short ( (5<<10) ); // hold time
	write_short ( (1<<12) ); // fade type
	write_byte ( 255 );
	write_byte ( 0 );
	write_byte ( 0 );
	write_byte ( 170 );
	message_end ( );
}

public goodeffect ( id ) {
	new gMsgScreenFade = get_user_msgid ( "ScreenFade" );
	message_begin ( MSG_ONE_UNRELIABLE , gMsgScreenFade , {0,0,0} , id );
	write_short ( (6<<10) ); // duration
	write_short ( (5<<10) ); // hold time
	write_short ( (1<<12) ); // fade type
	write_byte ( 0 );
	write_byte ( 0 );
	write_byte ( 255 );
	write_byte ( 170 );
	message_end ( );
}

public GiveYellowBonusBox ( id ) {
	
	switch (random_num(1,5)) 
	{
		case 1:
		{
			goodeffect ( id );
			set_user_health ( id, get_user_health ( id ) + 150 ); 
			ColorChat ( id, GREEN, "%s Ai primit^4 150^3 HP .^4", szPrefix );
		}
		
		case 2:
		{
			goodeffect ( id );
			set_user_armor ( id, get_user_armor ( id ) + 200 );
			ColorChat ( id, GREEN, "%s Ai Primit^4 200^3 AP .^4", szPrefix );
		}
		
		case 3:
		{
			if ( get_user_team ( id ) == 2 ) {
				
				dual_mp5 [ id ] = true;
				k1ases_weapon [ id ] = false;
				remove_dragoncannon ( id );
				give_item ( id, "weapon_mp5navy" );
				cs_set_user_bpammo ( id, CSW_MP5NAVY, 200 );
				ColorChat ( id, GREEN, "%s Ai primit^4 Dual Mp5^3 .^4", szPrefix );
				
			}
			
			else if ( get_user_team ( id ) == 1 ) {
				
				give_item ( id, "weapon_flashbang" );
				cs_set_user_bpammo ( id, CSW_FLASHBANG, 5 );
				ColorChat ( id, GREEN, "%s Ai primit^4 5^3 grenazi^4 FLASH^3 .^4", szPrefix );
				
			}
			
		}
		
		case 4:
		{
			give_item ( id, "weapon_hegrenade" );
			cs_set_user_bpammo ( id, CSW_HEGRENADE, 3 );
			ColorChat ( id, GREEN, "%s Ai primit^4 3^3 grenazi^4 HE^3 .^4", szPrefix );
		}
		
		case 5:
		{
			set_user_credits ( id, get_user_credits ( id ) + 5 );
			ColorChat ( id, GREEN, "%s Ai primit^4 5^3 credite .^4", szPrefix );
		}
	}
}

public GiveBonusBox(id) {
	
	if ( get_user_team ( id ) == 1 ) {
		
		switch (random_num(1,20)) 
		{
			case 1:
			{
				goodeffect ( id );
				set_user_health ( id, get_user_health ( id ) + 50 ); 
				ColorChat ( id, GREEN, "%s Ai primit^4 50^3 HP .^4", szPrefix );
			}
			
			case 2:
			{
				goodeffect ( id );
				set_user_armor ( id, get_user_armor ( id ) + 100 );
				ColorChat ( id, GREEN, "%s Ai primit^4 100^3 AP .^4", szPrefix );
			}
			
			case 3:
			{
				goodeffect ( id );
				set_user_health ( id, get_user_health ( id ) - 20 ); 
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 20^3 HP .^4", szPrefix );
			}
			
			case 4:
			{
				goodeffect ( id );
				set_user_armor ( id, get_user_armor ( id ) - 50 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 50^3 AP .^4", szPrefix );
			}
			
			case 5:
			{
				goodeffect ( id );
				cs_set_user_money ( id, cs_get_user_money ( id ) + 3000 );
				ColorChat ( id, GREEN, "%s Ai primit^4 3000^3 $ .^4", szPrefix );
			}
			
			case 6:
			{
				cs_set_user_money ( id, cs_get_user_money ( id ) - 2000 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 2000^3 $ .^4", szPrefix );
			}
			
			case 7:
			{
				goodeffect ( id );
				set_user_gravity ( id, 0.4 );
				ColorChat ( id, GREEN, "%s Ai primit^4 400^3 Gravity .^4", szPrefix );
			}
			
			case 8:
			{
				goodeffect ( id );
				HasSpeed[id] = true;
				ColorChat ( id, GREEN, "%s Ai primit^4 1000^3 Speed .^4", szPrefix );
			}
			
			case 9:
			{
				LowSpeed [ id ] = true;
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut din viteza .^4", szPrefix );
			}
			
			case 10:
			{
				set_user_health ( id, 225 );
				set_user_armor ( id, 225 );
				goodeffect ( id );
				ColorChat ( id, GREEN, "%s Acum ai^4 225^3 HP si^4 225^3 AP^4 .^4", szPrefix );
			}
			
			case 11:
			{
				cs_set_user_money ( id, 0 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut toti banii !^4", szPrefix );
			}
			
			case 12:
			{
				
				if ( !super_knife_shop [ id ] || !super_knife_shop2 [ id ] ) {
					
					goodeffect ( id );
					if ( Level [ id ] < 15 ) {
						super_knife_shop [ id ] = true;
						set_pev ( id, pev_viewmodel2, super_knife_shop_v_model );
					}
					
					else if ( Level [ id ] >= 15 ) {
						super_knife_shop2 [ id ] = true;
						set_pev ( id, pev_viewmodel2, super_knife_shop_v_model2 );
						set_pev ( id, pev_weaponmodel2, super_knife_shop_p_model2 );
					}
					
					ColorChat ( id, GREEN, "%s Ai primit^4 Super Knife^4 .^4", szPrefix ); 
					
				}
				
				else if ( super_knife_shop [ id ] || super_knife_shop2 [ id ] ) {
					goodeffect ( id );
					fm_set_user_health ( id, get_user_health ( id ) + 70 );
					ColorChat ( id, GREEN, "%s Ai primit^4 70^3 HP .^4", szPrefix );
				}
			}
			
			case 13:
			{
				goodeffect ( id );
				set_user_xp ( id, get_user_xp ( id ) + 100 );
				ColorChat ( id, GREEN, "%s Ai primit^4 100^3 XP .^4", szPrefix );
			}
			
			case 14:
			{
				wrongeffect ( id );
				set_user_xp ( id, get_user_xp ( id ) - 50 );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 50^3 XP .^4", szPrefix );
			}
			
			case 15:
			{
				ColorChat ( id, GREEN, "%s A trecut mosul pe la tine, ai primit^4 16000^3 $ .^4", szPrefix );
				goodeffect ( id );
			}
			
			case 16:
			{	
				if ( !vip_axe_knife [ id ] || !deklowaz [ id ] ) {
					HasTeleport[id] = true;
					client_cmd(id, "bind x power2");
					goodeffect ( id );
					ColorChat ( id, GREEN, "%s Ai primit puterea de a te teleporta, apasa tasta^4 X^3 pentru a te teleporta .^4", szPrefix );
				}
				
				else {
					set_user_credits ( id, get_user_credits ( id ) + 2 );
					ColorChat ( id, GREEN, "%s Ai primit^4 2^3 credite .", szPrefix );
				}
				
				
			}
			
			case 17:
			{
				
				HasSpeed[id] = true;
				client_cmd(id, "cl_sidespeed %d",get_pcvar_float(CvarFurienSpeed))
				client_cmd(id, "cl_forwardspeed %d",get_pcvar_float(CvarFurienSpeed))
				client_cmd(id, "cl_backspeed %d",get_pcvar_float(CvarFurienSpeed))
				set_user_maxspeed(id, get_pcvar_float(CvarFurienSpeed));
			}
			
			case 18:
			{
				
				
				fm_set_user_health ( id, get_user_health ( id ) + 80 );
				ColorChat ( id, GREEN, "%s Ai primit^4 80^3 HP .^4", szPrefix );
			}
			
			case 19:
			{
				give_item ( id, "weapon_hegrenade" );
				ColorChat ( id, GREEN, "%s Ai primit o grenada^4 HE^3 .", szPrefix );
			}
			
			case 20:
			{
				give_item ( id, "weapon_flashbang" );
				ColorChat ( id, GREEN, "%s Ai primit o grenada^4 FLASH^3 .", szPrefix );
			}
		}
	}
	
	else if ( get_user_team ( id ) == 2 ) {
		
		switch (random_num(1,23)) 
		{
			case 1:
			{
				goodeffect ( id );
				set_user_health ( id, get_user_health ( id ) + 50 ); 
				ColorChat ( id, GREEN, "%s Ai primit^4 50^3 HP .^4", szPrefix );
			}
			
			case 2:
			{
				goodeffect ( id );
				set_user_armor ( id, get_user_armor ( id ) + 100 );
				ColorChat ( id, GREEN, "%s Ai primit^4 100^3 AP .^4", szPrefix );
			}
			
			case 3:
			{
				goodeffect ( id );
				set_user_health ( id, get_user_health ( id ) - 20 ); 
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 20^3 HP .^4", szPrefix );
			}
			
			case 4:
			{
				goodeffect ( id );
				set_user_armor ( id, get_user_armor ( id ) - 50 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 50^3 AP .^4", szPrefix );
			}
			
			case 5:
			{
				goodeffect ( id );
				cs_set_user_money ( id, cs_get_user_money ( id ) + 3000 );
				ColorChat ( id, GREEN, "%s Ai primit^4 3000^3 $ .^4", szPrefix );
			}
			
			case 6:
			{
				cs_set_user_money ( id, cs_get_user_money ( id ) - 2000 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 2000^3 $ .^4", szPrefix );
			}
			
			case 7:
			{
				goodeffect ( id );
				set_user_gravity ( id, 0.4 );
				ColorChat ( id, GREEN, "%s Ai primit^4 400^3 Gravity .^4", szPrefix );
			}
			
			case 8:
			{
				goodeffect ( id );
				HasSpeed[id] = true;
				ColorChat ( id, GREEN, "%s Ai primit^4 750^3 Speed .^4", szPrefix );
			}
			
			case 9:
			{
				entity_set_float ( id, EV_FL_maxspeed, 250.0 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut din viteza .^4", szPrefix );
			}
			
			case 10:
			{
				set_user_health ( id, 10 );
				set_user_armor ( id, 20 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Acum ai^4 10^3 HP si^4 20^3 AP^4 .^4", szPrefix );
			}
			
			case 11:
			{
				cs_set_user_money ( id, 0 );
				wrongeffect ( id );
				ColorChat ( id, GREEN, "%s Ai pierdut toti banii !^4", szPrefix );
			}
			
			case 12:
			{
				
				if (  !salamander [ id ] ) {
					
					goodeffect ( id );
					salamander [ id ] = true;
					set_task ( 0.1, "SalamanderGiveItem", id );
					set_task ( 30.0, "reverse_salamander", id );
					ColorChat ( id, GREEN, "%s Ai primit^4 Salamander^3 pentru^4 30^3 secunde .", szPrefix );
				}
				
				else if ( salamander [ id ] ) {
					goodeffect ( id );
					fm_set_user_health ( id, get_user_health ( id ) + 100 );
					ColorChat ( id, GREEN, "%s Ai primit^4 100^3 HP .^4", szPrefix );
				}
				
			}
			
			case 13:
			{
				goodeffect ( id );
				set_user_xp ( id, get_user_xp ( id ) + 100 );
				ColorChat ( id, GREEN, "%s Ai primit^4 100^3 XP .^4", szPrefix );
			}
			
			case 14:
			{
				wrongeffect ( id );
				set_user_xp ( id, get_user_xp ( id ) - 20 );
				ColorChat ( id, GREEN, "%s Ai pierdut^4 20^3 XP .^4", szPrefix );
			}
			
			case 15:
			{
				ColorChat ( id, GREEN, "%s A trecut mosul pe la tine, ai primit^4 16000^3 $ .^4", szPrefix );
				goodeffect ( id );
			}
			
			case 16:
			{	
				if ( !vip_axe_knife [ id ] || !deklowaz [ id ] ) {
					HasTeleport[id] = true;
					client_cmd(id, "bind x power2");
					goodeffect ( id );
					ColorChat ( id, GREEN, "%s Ai primit puterea de a te teleporta, apasa tasta^4 X^3 pentru a te teleporta .^4", szPrefix );
				}
				
				else {
					goodeffect ( id );
					set_user_credits ( id, get_user_credits ( id ) + 2 );
					ColorChat ( id, GREEN, "%s Ai primit^4 2^3 credite .", szPrefix );
				}
				
				
			}
			
			case 17:
			{
				goodeffect ( id );
				HasSpeed[id] = true;
				client_cmd(id, "cl_sidespeed %d",get_pcvar_float(CvarAntiFurienSpeed))
				client_cmd(id, "cl_forwardspeed %d",get_pcvar_float(CvarAntiFurienSpeed))
				client_cmd(id, "cl_backspeed %d",get_pcvar_float(CvarAntiFurienSpeed))
				set_user_maxspeed(id, get_pcvar_float(CvarAntiFurienSpeed));
			}
			
			
			case 18:
			{
				
				if ( !k1ases_weapon [ id ] ) {
					goodeffect ( id );
					k1ases_weapon [ id ] = true;
					dual_mp5 [ id ] = false;
					remove_dragoncannon ( id );
					give_k1ases ( id );
					ColorChat ( id, GREEN, "%s Ai primit^4 K1ASUS^3 .^4", szPrefix );
				}
				
				else if ( k1ases_weapon [ id ] ) {
					goodeffect ( id );
					fm_set_user_health ( id, get_user_health ( id ) + 70 );
					ColorChat ( id, GREEN, "%s Ai primit^4 70^3 HP .^4", szPrefix );
				}
				
				
			}
			
			case 19:
			{
				goodeffect ( id );
				give_item ( id, "weapon_hegrenade" );
				ColorChat ( id, GREEN, "%s Ai primit o grenada^4 HE^3 .", szPrefix );
			}
			
			case 20:
			{
				goodeffect ( id );
				give_item ( id, "weapon_flashbang" );
				ColorChat ( id, GREEN, "%s Ai primit o grenada^4 FLASH^3 .", szPrefix );
			}
			
			case 21:
			{
				if ( UserHaveQuad [ id ] ) {
					goodeffect ( id );
					ColorChat ( id, GREEN, "%s Ai primit^4 5^3 gloante pentru^4 Quad-Barrel^3  .^4", szPrefix );
					cs_set_user_bpammo ( id, CSW_QB, cs_get_user_bpammo ( id, CSW_QB ) + 5 );
				}
				
				else {
					goodeffect ( id );
					set_task ( 0.1, "GiveQuadBarrel", id );
					ColorChat ( id, GREEN, "%s Ai primit^4 Quad-Barrel^3 .", szPrefix );
					UserHaveQuad [ id ] = true;
				}
			}
			
			case 22:
			{
				if ( UserHaveDragon [ id ] ) {
					goodeffect ( id );
					g_cannon_ammo [ id ] += 3;
					ColorChat ( id, GREEN, "%s Ai primit^4 3^3 gloante pentru^4 Dragon Cannon^3  .^4", szPrefix );
				}
				else {
					goodeffect ( id );
					ColorChat ( id, GREEN, "%s Ai primit^4 Dragon Cannon^3 .", szPrefix );
					set_task ( 0.1, "get_dragoncannon", id );
					UserHaveDragon [ id ] = true;
				}
			}
			
			case 23:
			{
				if ( UserHaveM79  [ id ] ) {
					goodeffect ( id );
					ColorChat ( id, GREEN, "%s Ai primit^4 2^3 grenazi pentru lansator. ^4", szPrefix );
					grenade_count [ id ] += 2;
				}
				
				else {
					goodeffect ( id );
					ColorChat ( id, GREEN, "%s Ai primit un^4 Lansator de Grenazi^3 .^4", szPrefix );
					give_weapon ( id, 2 );
					m79++;
					UserHaveM79 [ id ] = true;
				}
			}
			
			
		}
	}
}

public CmdTeleport(id) {
	if (!is_user_alive(id) || !HasTeleport[id]) return PLUGIN_CONTINUE
	
	if (Teleport_Cooldown[id]) {
		ColorChat ( id, GREEN, "%s Puterea iti va reveni in^4 %d^3 secunde .", Teleport_Cooldown [ id ] );
		return PLUGIN_CONTINUE
	}
	else if (teleport(id)) {
		emit_sound(id, CHAN_STATIC, SOUND_BLINK, 1.0, ATTN_NORM, 0, PITCH_NORM)
		remove_task(id)
		Teleport_Cooldown[id] = get_pcvar_num(CvarTeleportCooldown);
		set_task(1.0, "TeleportShowHUD", id, _, _, "b");
		set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
		if(get_pcvar_num(CvarTeleportCooldown) != 1) {
			show_hudmessage(id, "Puterea iti va reveni in^4 %d^3 secunde",get_pcvar_num(CvarTeleportCooldown));
		}
		if(get_pcvar_num(CvarTeleportCooldown) == 1) {
			show_hudmessage(id, "Puterea iti va reveni in^4 %d^3 secunda",get_pcvar_num(CvarTeleportCooldown));
		}
	}
	else {
		ColorChat ( id, GREEN, "%s Pozitia de teleportare este invalida .^4", szPrefix ); 
	}
	return PLUGIN_CONTINUE
}

public Light(entity, red, green, blue)
{	
	if(is_valid_ent(entity)) {
		static Float:origin[3]
		pev(entity, pev_origin, origin)
		
		message_begin(MSG_BROADCAST, SVC_TEMPENTITY, _, entity);
		write_byte(TE_DLIGHT) // TE id
		engfunc(EngFunc_WriteCoord, origin[0])
		engfunc(EngFunc_WriteCoord, origin[1])
		engfunc(EngFunc_WriteCoord, origin[2])
		write_byte(7) 
		write_byte(red)
		write_byte(green)
		write_byte(blue)
		write_byte(2)
		write_byte(0)
		message_end();
	}
}

public cmdVipWeaponsMenu ( id, level, cid ) { 
	
	new menu = menu_create ( "\rFurien XP \yMenu", "VIPWeapons_Giver" );
	
	if ( get_user_team ( id ) == 2 ) {
		
		menu_additem ( menu, "\ySCORPION MP5 [ \r3 CREDITE \y]", "1", VIP_ACCESS );
		menu_additem ( menu, "\yQUAD BARREL [ \r3 CREDITE \y]", "2", VIP_ACCESS );
		menu_additem ( menu, "\yDRAGON CANNON [ \r4 CREDITE \y]", "3", VIP_ACCESS );
		menu_additem ( menu, "\yM79 LAUNCHER [ \r4 CREDITE \y]", "4", VIP_ACCESS );
		
		
		menu_setprop ( menu, MPROP_EXIT, MEXIT_ALL );
		menu_display ( id, menu, 0 );
		
	}
	
	return 1;
}

public VIPWeapons_Giver ( id, menu, item ) {
	
	if( item == MENU_EXIT )
	{
		return 1;
	}
	
	new data [ 6 ], szName [ 64 ];
	new access, callback;
	menu_item_getinfo ( menu, item, access, data,charsmax ( data ), szName,charsmax ( szName ), callback );
	new key = str_to_num ( data );
	
	switch(key)
	{
		case 1:
		{
			new iCredits = get_user_credits(id) - 3;
			if( iCredits < 0 )
			{
				ColorChat ( id, GREEN, "%s Nu ai destule credite !", szPrefix );
				return 1;
			}
			else
			{
				ColorChat ( id, GREEN, "%s Ai cumparat^4 Scorpion Mp5^3 .^4", szPrefix );
				give_k1ases ( id );
				set_user_credits ( id, iCredits );
				return 1;
			}
		}
		
		case 2:
		{
			new iCredits = get_user_credits(id) - 3;
			if( iCredits < 0 )
			{
				ColorChat ( id, GREEN, "%s Nu ai destule credite !", szPrefix );
				return 1;
			}
			else
			{
				ColorChat ( id, GREEN, "%s Ai cumparat^4 Quad-Barrel^3 .^4", szPrefix );
				set_task ( 0.1, "GiveQuadBarrel", id );
				set_user_credits ( id, iCredits );
				return 1;
			}
		}
		
		case 3:
		{
			new iCredits = get_user_credits(id) - 4;
			if( iCredits < 0 )
			{
				ColorChat ( id, GREEN, "%s Nu ai destule credite !", szPrefix );
				return 1;
			}
			else
			{
				ColorChat ( id, GREEN, "%s Ai cumparat^4 Dragon Cannon^3 .^4", szPrefix );
				set_task ( 0.1, "get_dragoncannon", id );
				set_user_credits ( id, iCredits );
				return 1;
			}
		}
		
		case 4:
		{
			new iCredits = get_user_credits(id) - 4;
			if( iCredits < 0 )
			{
				ColorChat ( id, GREEN, "%s Nu ai destule credite !", szPrefix );
				return 1;
			}
			else
			{
				ColorChat ( id, GREEN, "%s Ai cumparat^4 M79 Launcher^3 .^4", szPrefix );
				give_weapon ( id, 2 );
				m79++;
				UserHaveM79 [ id ] = true;
				set_user_credits ( id, iCredits );
				return 1;
			}
		}
		
	}
	
	menu_destroy ( menu );
	return 1;
	
}

public cmdXpMenu ( id, level, cid ) { 
	
	new menu = menu_create ( "\rFurien XP \yMenu", "Menu_Giver" );
	menu_additem ( menu, "\y3 HE \y[ \rLvL. 3 - 1 Credit\y ]", "1", 0 );
	menu_additem ( menu, "\y5 FB & 2 SMK \y[ \rLvL. 5 - 1 Credit \y ]", "2", 0 );
	menu_additem ( menu, "\y6 HE \y[ \rLvL. 8 - 2 Credite\y ]", "3", 0 );
	menu_additem ( menu, "\ySPEED \y[ \rLvL. 10 - 2 Credite \y ]", "4", 0 );
	menu_additem ( menu, "\yGRAVITY \y[ \rLvL. 13 - 2 Credite\y ]", "5", 0 );
	menu_additem ( menu, "\yGODMODE \y[ \rLvL. 16 - 3 Credite\y ]", "6", 0 );
	menu_additem ( menu, "\yNOCLIP \y[ \rLvL. 18 - 3 Credite\y ]", "7", 0 );
	menu_additem ( menu, "\yCHAMELEON \y[ \rLvL. 22 - 4 Credite \y ]", "8", 0 );
	menu_additem ( menu, "\yRESPAWN \y[ \rLvL. 25 - 5 Credite \y ]", "9", 0 );
	
	if ( get_user_team ( id ) == 2 ) {
		menu_additem ( menu, "\ySALAMANDER \y[ \rLvL. 28 - 5 Credite \y ]", "10", 0 );
	}
	
	menu_setprop ( menu, MPROP_EXIT, MEXIT_ALL );
	menu_display ( id, menu, 0 );
	
	
	return 1;
}

public Menu_Giver ( id, menu, item ) {
	
	if( item == MENU_EXIT )
	{
		return 1;
	}
	
	if ( g_iCount [ id ] >= 1 ) {
		ColorChat ( id, GREEN, "%s Ai folosit odata aceasta optiune .", szPrefix );
	}
	
	new data [ 6 ], szName [ 64 ];
	new access, callback;
	menu_item_getinfo ( menu, item, access, data,charsmax ( data ), szName,charsmax ( szName ), callback );
	new key = str_to_num ( data );
	
	switch(key)
	{
		case 1:
		{
			if ( g_Menu [ id ] >= 3 ) {
				
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				
				if ( Level [ id ] >= 3 ) {
					give_item ( id, "weapon_hegrenade" );
					cs_set_user_bpammo ( id, CSW_HEGRENADE, 3 );
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 1 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 3 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 2:
		{
			
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 5 ) {
					give_item ( id, "weapon_flashbang" );
					give_item ( id, "weapon_smokegrenade" );
					cs_set_user_bpammo ( id, CSW_FLASHBANG, 5 );
					cs_set_user_bpammo ( id, CSW_SMOKEGRENADE, 2 );
					set_user_credits ( id, get_user_credits ( id ) - 1 );
					++g_Menu [ id ];
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 5 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 3:
		{
			
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 8 ) {
					give_item ( id, "weapon_hegrenade" );
					cs_set_user_bpammo ( id, CSW_HEGRENADE, 6 );
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 2 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 8 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 4:
		{
			
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 10 ) {
					entity_set_float ( id, EV_FL_maxspeed, 450.0 );  
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 2 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 10 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 5:
		{
			
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 13 ) {
					set_user_gravity ( id, 0.5 );
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 2 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 13 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 6:
		{
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 16 ) {
					set_user_godmode ( id, 1 );
					ColorChat ( id, GREEN, "%s Ai primit God pentru^4 8^4 secunde .", szPrefix );
					set_task ( 8.0, "reverse_godmode", id );
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 3 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 16 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 7:
		{
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 18 ) {
					set_user_noclip ( id, 1 );
					ColorChat ( id, GREEN, "%s Ai primit Noclip pentru^4 5^4 secunde .", szPrefix );
					set_task ( 5.0, "reverse_noclip", id );
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 3 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 18 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 8:
		{
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
				return 1;
			}
			
			else {
				if ( Level [ id ] >= 22 ) {
					chameleon ( id );
					++g_Menu [ id ];
					set_user_credits ( id, get_user_credits ( id ) - 4 );
					return 1;
				}
				
				else {
					ColorChat ( id, GREEN, "%s Nu ai level 22 .", szPrefix );
					return 1;
				}
			}
		}
		
		case 9:
		{
			if ( g_Menu [ id ] >= 3 ) {
				ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
			}
			
			else {
				if ( Level [ id ] >= 25 ) {
					
					if(is_user_alive(id)) 
					{
						ColorChat ( id, GREEN, "%s Esti deja in viata .", szPrefix );
						return 1;
					}
					
					if ( get_user_team ( id ) == 3) {
						
						ColorChat ( id, GREEN, "%s Esti spectator .", szPrefix );
						
						if(!is_user_alive(id)) 
						{
							
							ExecuteHamB ( Ham_CS_RoundRespawn, id );
							set_task(0.5, "SetUserWeapons", id + 12345);
							++g_Menu [ id ];
							set_user_credits ( id, get_user_credits ( id ) - 5 );
							return 1;
						}
						
						return 1;
					}
					
					else {
						ColorChat ( id, GREEN, "%s Nu ai level 25 .", szPrefix );
						return 1;
					}
				}
			}
		}
		
		case 10:
		{
			if ( get_user_team ( id ) == 2 ) {
				
				if ( g_Menu [ id ] >= 3 ) {
					ColorChat ( id, GREEN, "%s Ai folosit deja meniul de^4 3^3 ori .", szPrefix );
					return 1;
				}
				
				else {
					if ( Level [ id ] >= 28 && !SalamanderLimit [ id ] ) {
						
						salamander [ id ] = true;
						ColorChat ( id, GREEN, "%s Ai primit aceasta arma pentru^4 1^4 minut .", szPrefix );
						set_task ( 0.1, "SalamanderGiveItem", id );
						set_task ( 60.0, "reverse_salamander", id );
						++g_Menu [ id ];
						SalamanderLimit [ id ] = true;
						set_user_credits ( id, get_user_credits ( id ) - 5 );
						return 1;
						
						
					}
					else if ( SalamanderLimit [ id ] ) {
						ColorChat ( id, GREEN, "%s Ai mai folosit odata aceasta optiune .", szPrefix );
						return 1;
					}
					
					else if ( Level [ id ] < 28 ) {
						ColorChat ( id, GREEN, "%s Nu ai level 30 .", szPrefix );
						return 1;
					}
				}
			}
			
		}
	}
	
	menu_destroy ( menu );
	return 1;
	
}

public SetUserWeapons(id) {
	id -= 12345;
	
	if( !is_user_connected(id) ) return PLUGIN_HANDLED;
	
	strip_user_weapons(id);
	give_item(id, "weapon_knife");
	set_task ( 0.1, "cmdClassMenu", id );
	
	return PLUGIN_CONTINUE;
}

public reverse_godmode ( id ) {
	
	set_user_godmode ( id , 0 );
}

public reverse_noclip ( id ) {
	
	set_user_noclip ( id, 0 );
}

public reverse_invis ( id ) {
	
	set_user_rendering( id, kRenderFxNone, 0, 0, 0, kRenderTransAlpha, 255 );
	ColorChat ( id, GREEN, "%s Timpul a expirat ! Acum esti din nou^4 vizibil^3 .", szPrefix );
}

public reverse_salamander ( id ) {
	
	strip_user_weapons ( id );
	give_item ( id, "weapon_knife" );
	salamander [ id ] = false;
	set_task ( 0.1, "cmdClassMenu", id );
	
	ColorChat ( id, GREEN, "%s Timpul a expirat ! Nu mai ai arma^4 salamander^3 .", szPrefix );
}

public show_salamander_icon ( id ) {
	if ( salamander [ id ] ) {
		if (!(pev(id,pev_button) & FL_ONGROUND))
		{    
			new iconstatus;
			iconstatus = get_user_msgid("StatusIcon");
			message_begin(MSG_ONE,iconstatus,{0,0,0},id);
			write_byte(1); // status (0=hide, 1=show, 2=flash)
			write_string("dmg_heat"); // sprite name
			write_byte(255); // red
			write_byte(0); // green
			write_byte(0); // blue
			message_end();
		}
		
	}
	
	else {
		if (!(pev(id,pev_button) & FL_ONGROUND))
		{    
			new iconstatus;
			iconstatus = get_user_msgid("StatusIcon");
			message_begin(MSG_ONE,iconstatus,{0,0,0},id);
			write_byte(0); // status (0=hide, 1=show, 2=flash)
			write_string("dmg_heat"); // sprite name
			write_byte(255); // red
			write_byte(0); // green
			write_byte(0); // blue
			message_end();
		}
	}
}

public reverse_model ( id ) {
	
	if ( get_user_team ( id ) == 1 ) {
		cs_set_user_model ( id, "guerilla" );
		ColorChat ( id, GREEN, "%s Timpul a expirat ! Acum arati din nou ca un^4 Furien^3 .", szPrefix );
	}
	
	else if ( get_user_team ( id ) == 2 ) {
		cs_set_user_model ( id, "gign" );
		ColorChat ( id, GREEN, "%s Timpul a expirat ! Acum arati din nou ca un^4 AntiFurien^3 .", szPrefix );
	}
}

public chameleon ( id ) {
	
	if ( get_user_team ( id ) == 1 ) {
		cs_set_user_model ( id, "gign" );
		ColorChat ( id, GREEN, "%s Acum arati ca un^4 AntiFurien^3 pentru^4 60^3 secunde .", szPrefix );
		set_task ( 60.0, "reverse_model", id );
	}
	
	else if ( get_user_team ( id ) == 2 ) {
		cs_set_user_model ( id, "guerilla" );
		ColorChat ( id, GREEN, "%s Acum arati ca un^4 Furien^3 pentru^4 60^3 secunde .", szPrefix );
		set_task ( 60.0, "reverse_model", id );
	}
}

public grenade_throw ( id, gid, wid ) {
	
	if ( strike_grenade [ id ] || strike_grenade2 [ id ] || strike_grenade3 [ id ] && get_user_weapon ( id ) == CSW_HEGRENADE && get_user_team ( id ) == 1 ) {
		new gtm = 1;
		if(!gtm) return;
		new r, g, b;
		switch(gtm)
		{
			case 1:
			{
				r = 255;
				g = 0;
				b = 0;
			}
		}
		message_begin(MSG_BROADCAST, SVC_TEMPENTITY);
		write_byte(TE_BEAMFOLLOW);
		write_short(gid);
		write_short(g_trail);
		write_byte(10);
		write_byte(5);
		write_byte(r);
		write_byte(g);
		write_byte(b);
		write_byte(192);
		message_end();
	}
}

// POWERS -------------------------------------

////////////////////////////////////////////////////////////////////////////////////////////////////////////
// Drop Enemy Weapon |
//==========================================================================================================	
public DropShowHUD(id) {
	if (!is_user_alive(id) || HasPower[id] != 4) {
		remove_task(id);
		Drop_Cooldown[id] = 0;
		return PLUGIN_HANDLED;
	}
	set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
	if(is_user_alive(id) && Drop_Cooldown[id] == 1) {
		Drop_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunda",Drop_Cooldown[id]);
	}
	if(is_user_alive(id) && Drop_Cooldown[id] > 1) {
		Drop_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunde",Drop_Cooldown[id]);
	}
	if(Drop_Cooldown[id] <= 0) {
		show_hudmessage(id, "Ti-a revenit puterea");
		ColorChat(id, GREEN,"%s Iti poti folosi din nou puterea .", szPrefix);
		remove_task(id);
		Drop_Cooldown[id] = 0;
	}
	return PLUGIN_HANDLED;
}


////////////////////////////////////////////////////////////////////////////////////////////////////////////
// Freeze |
//==========================================================================================================
public Freeze(id) {
	if (!is_user_alive(id) || Frozen[id]) return;
	
	pev(id, pev_maxspeed, TempSpeed[id]); //get temp speed
	pev(id, pev_gravity, TempGravity[id]); //get temp speed
	fm_set_rendering(id, kRenderFxGlowShell, 0, 100, 200, kRenderNormal, 25);
	engfunc(EngFunc_EmitSound, id, CHAN_BODY, FROSTPLAYER_SND[random_num(0, sizeof FROSTPLAYER_SND - 1)], 1.0, ATTN_NORM, 0, PITCH_NORM);
	message_begin(MSG_ONE_UNRELIABLE, get_user_msgid("ScreenFade"), _, id);
	write_short(UNIT_SECOND*1);
	write_short(floatround(UNIT_SECOND*get_pcvar_float(CvarFreezeDuration)));
	write_short(FFADE_IN);
	write_byte(0);
	write_byte(50);
	write_byte(200);
	write_byte(100);
	message_end();
	if (pev(id, pev_flags) & FL_ONGROUND)
		set_pev(id, pev_gravity, 999999.9);
	else
		set_pev(id, pev_gravity, 0.000001);
	
	Frozen[id] = true;
	set_task(get_pcvar_float(CvarFreezeDuration), "remove_freeze", id);
}

public set_normal(id) {
	set_pev(id, pev_gravity, TempGravity[id]);
	set_pev(id, pev_maxspeed, TempSpeed[id]);
}

public FreezeShowHUD(id) {
	if (!is_user_alive(id) || HasPower[id] != 5) {
		remove_task(id);
		Freeze_Cooldown[id] = 0;
		return PLUGIN_HANDLED;
	}
	set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
	if(is_user_alive(id) && Freeze_Cooldown[id] == 1) {
		Freeze_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunda",Freeze_Cooldown[id]);
	}
	if(is_user_alive(id) && Freeze_Cooldown[id] > 1) {
		Freeze_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunde",Freeze_Cooldown[id]);
	}
	if(Freeze_Cooldown[id] <= 0) {
		show_hudmessage(id, "Ti-a revenit puterea");
		ColorChat(id, GREEN, "%s Iti poti folosi din nou puterea .", szPrefix);
		remove_task(id);
		Freeze_Cooldown[id] = 0;
	}
	return PLUGIN_HANDLED;
}

////////////////////////////////////////////////////////////////////////////////////////////////////////////
// Drag |
//==========================================================================================================
public DragStart(id) {
	if (HasPower[id] == 6 && !Drag_I[id]) {
		
		if (!is_user_alive(id)) {
			return PLUGIN_HANDLED;
		}
		if (Drag_Cooldown[id]) {
			ColorChat(id, GREEN,"%s Puterea iti va reveni in^4 %d^3 secunde .^4", szPrefix, Drag_Cooldown[id]);
			return PLUGIN_HANDLED;
		}
		new hooktarget, body;
		get_user_aiming(id, hooktarget, body);
		
		if (is_user_alive(hooktarget)) {
			if (get_user_team(id) != get_user_team(hooktarget)) {				
				Hooked[id] = hooktarget;
				emit_sound(hooktarget, CHAN_BODY, DRAG_HIT_SND, 1.0, ATTN_NORM, 0, PITCH_HIGH);
			}
			else {
				return PLUGIN_HANDLED;
			}
			
			if (get_pcvar_float(CvarDragSpeed) <= 0.0)
				CvarDragSpeed = 1;
			
			new parm[2];
			parm[0] = id;
			parm[1] = hooktarget;
			
			set_task(0.1, "DragReelin", id, parm, 2, "b");
			HarpoonTarget(parm);
			Drag_I[id] = true;
			Not_Cooldown[id] = false;
			if(get_pcvar_num(CvarDragUnb2Move) == 1)
				Unable2move[hooktarget] = true;
			
			if(get_pcvar_num(CvarDragUnb2Move) == 2)
				Unable2move[id] = true;
			
			if(get_pcvar_num(CvarDragUnb2Move) == 3) {
				Unable2move[hooktarget] = true;
				Unable2move[id] = true;
			}
		} 
		else {
			Hooked[id] = 33;
			NoTarget(id);
			Not_Cooldown[id] = false;
			set_task(1.0,"DragEnd",id);
			emit_sound(id, CHAN_BODY, DRAG_MISS_SND, 1.0, ATTN_NORM, 0, PITCH_HIGH);
			Drag_I[id] = true;
		}
	}
	else
		return PLUGIN_HANDLED;
	
	return PLUGIN_CONTINUE;
}

public DragShowHUD(id) {
	if (!is_user_alive(id) || HasPower[id] != 6) {
		remove_task(id);
		Drag_Cooldown[id] = 0;
		Not_Cooldown[id] = true;
		return PLUGIN_HANDLED;
	}
	set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
	if(is_user_alive(id) && Drag_Cooldown[id] == 1) {
		Drag_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunda",Drag_Cooldown[id]);
	}
	if(is_user_alive(id) && Drag_Cooldown[id] > 1) {
		Drag_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunde",Drag_Cooldown[id]);
	}
	if(Drag_Cooldown[id] <= 0) {
		show_hudmessage(id, "Ti-a revenit puterea");
		ColorChat(id, GREEN, "%s Iti poti folosi din nou puterea .", szPrefix);
		remove_task(id);
		Drag_Cooldown[id] = 0;
		Not_Cooldown[id] = true;
	}
	return PLUGIN_HANDLED;
}

public DragReelin(parm[]) {
	new id = parm[0];
	new victim = parm[1];
	
	if (!Hooked[id] || !is_user_alive(victim)) {
		DragEnd(id);
		return;
	}
	
	new Float:fl_Velocity[3];
	new idOrigin[3], vicOrigin[3];
	
	get_user_origin(victim, vicOrigin);
	get_user_origin(id, idOrigin);
	
	new distance = get_distance(idOrigin, vicOrigin);
	
	if (distance > 1) {
		new Float:fl_Time = distance / get_pcvar_float(CvarDragSpeed);
		
		fl_Velocity[0] = (idOrigin[0] - vicOrigin[0]) / fl_Time;
		fl_Velocity[1] = (idOrigin[1] - vicOrigin[1]) / fl_Time;
		fl_Velocity[2] = (idOrigin[2] - vicOrigin[2]) / fl_Time;
	}
	else {
		fl_Velocity[0] = 0.0;
		fl_Velocity[1] = 0.0;
		fl_Velocity[2] = 0.0;
	}
	
	entity_set_vector(victim, EV_VEC_velocity, fl_Velocity); //<- rewritten. now uses engine
}

public TakeDamage(victim, inflictor, attacker, Float:damage) { // if take damage drag off
	if (is_user_alive(attacker) && (get_pcvar_num(CvarDragDmg2Stop) > 0)) {
		OvrDmg[victim] = OvrDmg[victim] + floatround(damage);
		if (OvrDmg[victim] >= get_pcvar_num(CvarDragDmg2Stop)) {
			OvrDmg[victim] = 0;
			DragEnd(victim);
			return HAM_IGNORED;
		}
	}
	
	return HAM_IGNORED;
}

public HarpoonTarget(parm[]) { // set beam (ex. tongue:) if target is player
	
	new id = parm[0];
	new hooktarget = parm[1];
	
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY);
	write_byte(8);	// TE_BEAMENTS
	write_short(id);
	write_short(hooktarget);
	write_short(DragSprite);	// sprite index
	write_byte(0);	// start frame
	write_byte(0);	// framerate
	write_byte(200);	// life
	write_byte(8);	// width
	write_byte(1);	// noise
	write_byte(155);	// r, g, b
	write_byte(155);	// r, g, b
	write_byte(55);	// r, g, b
	write_byte(90);	// brightness
	write_byte(10);	// speed
	message_end();
}

public NoTarget(id) { // set beam if target isn't player
	new endorigin[3];
	
	get_user_origin(id, endorigin, 3);
	
	message_begin(MSG_BROADCAST, SVC_TEMPENTITY);
	write_byte(TE_BEAMENTPOINT); // TE_BEAMENTPOINT
	write_short(id);
	write_coord(endorigin[0]);
	write_coord(endorigin[1]);
	write_coord(endorigin[2]);
	write_short(DragSprite); // sprite index
	write_byte(0);	// start frame
	write_byte(0);	// framerate
	write_byte(200);	// life
	write_byte(8);	// width
	write_byte(1);	// noise
	write_byte(155);	// r, g, b
	write_byte(155);	// r, g, b
	write_byte(55);	// r, g, b
	write_byte(75);	// brightness
	write_byte(0);	// speed
	message_end();
}

public PlayerPreThink(id) {
	new button = get_user_button(id);
	new oldbutton = get_user_oldbutton(id);
	
	if (!is_user_alive(id)) {
		return FMRES_IGNORED;
	}
	
	if (Frozen[id]) {
		set_pev(id, pev_velocity, Float:{0.0,0.0,0.0});
		set_pev(id, pev_maxspeed, 1.0) ;
	}
	
	if(HasPower[id] == 6 ) { 
		if (BindUse[id]) {
			if (!(oldbutton & IN_USE) && (button & IN_USE))
				DragStart(id);
			
			if ((oldbutton & IN_USE) && !(button & IN_USE))
				DragEnd(id);
		}
		
		if (!Drag_I[id]) {
			Unable2move[id] = false;
		}
		
		if (Unable2move[id] && get_pcvar_num(CvarDragUnb2Move) > 0) {
			set_pev(id, pev_maxspeed, 1.0);
		}
	}
	return PLUGIN_CONTINUE;
}

////////////////////////////////////////////////////////////////////////////////////////////////////////////
// Teleport |
//==========================================================================================================
public TeleportShowHUD(id) {
	if (!is_user_alive(id) || HasPower[id] != 7) {
		remove_task(id);
		Teleport_Cooldown[id] = 0;
		return PLUGIN_HANDLED;
	}
	set_hudmessage(0, 100, 200, 0.05, 0.60, 0, 1.0, 1.1, 0.0, 0.0, -11);
	if(is_user_alive(id) && Teleport_Cooldown[id] == 1) {
		Teleport_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunda",Teleport_Cooldown[id]);
	}
	if(is_user_alive(id) && Teleport_Cooldown[id] > 1) {
		Teleport_Cooldown[id] --;
		show_hudmessage(id, "Puterea iti va reveni in: %d secunde",Teleport_Cooldown[id]);
	}
	if(Teleport_Cooldown[id] <= 0) {
		show_hudmessage(id, "Ti-a revenit puterea");
		ColorChat(id, GREEN, "%s Iti poti folosi din nou puterea. ", szPrefix);
		remove_task(id);
		Teleport_Cooldown[id] = 0;
	}
	return PLUGIN_HANDLED;
}





////////////////////////////////////////////////////////////////////////////////////////////////////////////
// NoRecoil |
//==========================================================================================================
public Weapon_PrimaryAttack_Pre(entity) {
	new id = pev(entity, pev_owner);
	
	if (HasPower[id] == 8) {
		pev(id, pev_punchangle, cl_pushangle[id]);
		return HAM_IGNORED;
	}
	return HAM_IGNORED;
}

public Weapon_PrimaryAttack_Post(entity) {
	new id = pev(entity, pev_owner);
	
	if ( HasPower[id] == 8) {
		new Float: push[3];
		pev(id, pev_punchangle, push);
		xs_vec_sub(push, cl_pushangle[id], push);
		xs_vec_mul_scalar(push, 0.0, push);
		xs_vec_add(push, cl_pushangle[id], push);
		set_pev(id, pev_punchangle, push);
		return HAM_IGNORED;
	}
	return HAM_IGNORED;
}

// POWERS -------------------------------------------------

public FurienAndAntiFurienDamage ( iVictim, iInflictor, iAttacker, Float:fDamage, iDamageBits )
{
	if( iInflictor == iAttacker && vip_axe_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 5.3);
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && katana_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 2.8);
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && double_katana_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 3.3);
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && super_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 2.0);
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && infinity_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 1.5 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && super_knife_shop [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 6.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && super_knife_shop2 [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 10.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && ignes_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 4.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && elf_knife [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_KNIFE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 4.5 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && hunter [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_P90 && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 1.5 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && mage [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_GALIL && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 2.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && rogue [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_FAMAS && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 2.5 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && shaman [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_SG552 && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 2.5 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && thompson [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_P90 && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 3.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && warrior [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_P90 && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 2.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && deklowaz [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_P90 && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 3.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && dual_mp5 [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_MP5NAVY && cs_get_user_team( iAttacker ) == CS_TEAM_CT )
	{
		SetHamParamFloat( 4, fDamage * 2.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && strike_grenade [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_HEGRENADE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 4.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && strike_grenade2 [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_HEGRENADE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 5.0 );
		return HAM_HANDLED;
	}
	
	if( iInflictor == iAttacker && strike_grenade3 [ iAttacker ] && is_user_alive( iAttacker ) && get_user_weapon( iAttacker ) == CSW_HEGRENADE && cs_get_user_team( iAttacker ) == CS_TEAM_T )
	{
		SetHamParamFloat( 4, fDamage * 6.0 );
		return HAM_HANDLED;
	}
	
	return HAM_IGNORED;
} 

public Depozit(id) {
	if(cs_get_user_money(id) >= 16000) {
		ColorChat ( id, GREEN, "%s Ai depozitat^4 16000^3 si ai primit^4 1^3 credit .^4", szPrefix );
		set_user_credits(id, get_user_credits(id) + 1);
		cs_set_user_money(id, cs_get_user_money(id) - 16000);
	}
	else {
		ColorChat ( id, GREEN, "%s Ai nevoie de ^4 16000 $^3 pentru a depozita .^4", szPrefix );
	}
	return PLUGIN_HANDLED;
}

public Retrage(id) {
	if(cs_get_user_money(id) >= 16000) {
		ColorChat ( id, GREEN, "%s Detii deja^4 16000 $^3", szPrefix );
	}
	else if(PlayerCredits[id]) {
		ColorChat ( id, GREEN, "%s Ai retras^4 1^3 credit, mai ai^4 %d^3 credite .", szPrefix, PlayerCredits [ id ] - 1 );
		set_user_credits(id, get_user_credits(id) - 1);
		cs_set_user_money(id, cs_get_user_money(id) + 16000);
	}
	else {
		ColorChat ( id, GREEN, "%s Ai nevoie de^4 1^3 credti pentru a putea retrage .^4", szPrefix );
	}
	return PLUGIN_HANDLED;
}

public Show_Credits(id) {
	set_hudmessage(0, 128, 0, 0.03, 0.86, 2, 6.0, 5.0);
	show_hudmessage(id, "Ai %d Credite.", PlayerCredits[id]);
	ColorChat ( id, GREEN, "%s Detii^4 %d^3 credite .^4", szPrefix, PlayerCredits [ id ] );
	return PLUGIN_HANDLED;
}

public Give_Credits(id, level, cid) {
	if(!cmd_access(id, level, cid, 2)) {
		return PLUGIN_HANDLED;
	}
	new arg[23], gplayers[32], num, i, players, name[32];
	get_user_name(id, name, 31);
	read_argv(1, arg, 23);
	new give_credits[5];
	read_argv(2, give_credits, charsmax(give_credits));
	new Credits = str_to_num(give_credits);
	if(equali(arg, "@T") || equali ( arg, "t" ) ) {
		get_players(gplayers, num, "e", "TERRORIST");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			set_user_credits(players, get_user_credits(players) + Credits);
			SaveData(id);
		}
		switch(get_cvar_num("amx_show_activity")) {
			case 1: ColorChat(0, GREEN, "^x03ADMIN^x04 give^x03 %i Credits^x04 to all^x03 Ts.", Credits);
				case 2: ColorChat(0, GREEN, "^x03%s^x04 give^x03 %i Credits^x04 to all^x03 Ts.", name, Credits);
			}
	}
	else if(equali(arg, "@CT") || equali ( arg, "ct" ) ) {
		get_players(gplayers, num, "e", "CT");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			set_user_credits(players, get_user_credits(players) + Credits);
			SaveData(id);
		}
		switch(get_cvar_num("amx_show_activity")) {
			case 1: ColorChat(0, GREEN, "^x03ADMIN^x04 give^x03 %i Credits^x04 to all^x03 CTs.", Credits);
				case 2: ColorChat(0, GREEN, "^x03%s^x04 give^x03 %i Credits^x04 to all^x03 CTs.", name, Credits);
			}
	}
	if(equali(arg, "@All") || equali ( arg, "all" ) ) {
		get_players(gplayers, num, "a");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			set_user_credits(players, get_user_credits(players) + Credits);
			SaveData(id);
		}
		switch(get_cvar_num("amx_show_activity")) {
			case 1: ColorChat(0, GREEN, "^x03ADMIN^x04 give^x03 %i Credits^x04 to all^x03 Players.", Credits);
				case 2: ColorChat(0, GREEN, "^x03%s^x04 give^x03 %i Credits^x04 to all^x03 Players.", name, Credits);
			}
	}
	new player = cmd_target(id, arg, 11);
	if(!player) {
		return PLUGIN_HANDLED;
	}
	set_user_credits(player, get_user_credits(player) + Credits);
	SaveData(id);
	switch(get_cvar_num("amx_show_activity")) {
		case 1: ColorChat(player, GREEN, "^x03ADMIN^x04 give your^x03 %i Credits.", Credits);
			case 2: ColorChat(player, GREEN, "^x03%s^x04 give your^x03 %i Credits.", name, Credits);
		}
	return PLUGIN_HANDLED;
}

public Reset_Credits(id, level, cid) {
	if(!cmd_access(id, level, cid, 2)) {
		return PLUGIN_HANDLED;
	}
	new arg[23], gplayers[32], num, i, players, name[32];
	get_user_name(id, name, 31);
	read_argv(1, arg, 23);
	if(equali(arg, "@T")) {
		get_players(gplayers, num, "e", "TERRORIST");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			PlayerCredits [ players ] = 0;
		}
		switch(get_cvar_num("amx_show_activity")) {
			case 1: ColorChat(0, GREEN, "^x03ADMIN^x04 reset^x03 Credits^x04 to all^x03 Ts.");
				case 2: ColorChat(0, GREEN, "^x03%s^x04 reset^x03 Credits^x04 to all^x03 Ts.", name);
			}
	}
	
	else if(equali(arg, "@CT")) {
		get_players(gplayers, num, "e", "CT");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			PlayerCredits [ players ] = 0;
		}
		switch(get_cvar_num("amx_show_activity")) {
			case 1: ColorChat(0, GREEN, "^x03ADMIN^x04 reset^x03 %i Credits^x04 to all^x03 CTs.");
				case 2: ColorChat(0, GREEN, "^x03%s^x04 reset^x03 %i Credits^x04 to all^x03 CTs.", name);
			}
	}
	if(equali(arg, "@All")) {
		get_players(gplayers, num, "a");
		for(i = 0; i < num; i++) {
			players = gplayers[i];
			if(!is_user_connected(players))
				continue;
			PlayerCredits [ players ] = 0;
		}
		switch(get_cvar_num("amx_show_activity")) {
			case 1: ColorChat(0, GREEN, "^x03ADMIN^x04 reset^x03 Credits^x04 to all^x03 Players.");
				case 2: ColorChat(0, GREEN, "^x03%s^x04 resetx03 Credits^x04 to all^x03 Players.", name);
			}
	}
	new player = cmd_target(id, arg, 11);
	if(!player) {
		return PLUGIN_HANDLED;
	}
	PlayerCredits [ player ] = 0;
	switch(get_cvar_num("amx_show_activity")) {
		case 1: ColorChat(player, GREEN, "^x03ADMIN^x04 reset your^x03 Credits.");
			case 2: ColorChat(player, GREEN, "^x03%s^x04 reset your^x03 Credits.", name);
		}
	return PLUGIN_HANDLED;
}

stock set_player_nextattack(player, weapon_id, Float:NextTime)
{
	const m_flNextPrimaryAttack_dc = 46
	const m_flNextSecondaryAttack_dc = 47
	const m_flTimeWeaponIdle_dc = 48
	const m_flNextAttack_dc = 83
	
	static weapon
	weapon = fm_get_user_weapon_entity(player, weapon_id)
	
	set_pdata_float(player, m_flNextAttack_dc, NextTime, 5)
	if(pev_valid(weapon))
	{
		set_pdata_float(weapon, m_flNextPrimaryAttack_dc , NextTime, 4)
		set_pdata_float(weapon, m_flNextSecondaryAttack_dc, NextTime, 4)
		set_pdata_float(weapon, m_flTimeWeaponIdle_dc, NextTime, 4)
	}
}

//get weapon id
stock get_weapon_ent(id,wpnid=0,wpnName[]="") {
	// who knows what wpnName will be
	static newName[24];
	
	// need to find the name
	if(wpnid) get_weaponname(wpnid,newName,23);
	
	// go with what we were told
	else formatex(newName,23,"%s",wpnName);
	
	// prefix it if we need to
	if(!equal(newName,"weapon_",7))
		format(newName,23,"weapon_%s",newName);
	
	return fm_find_ent_by_owner(get_maxplayers(),newName,id);
} 

// Blood and bodyparts
stock create_blood(const Float:origin[3]) {
	// Head
	message_begin(MSG_BROADCAST,SVC_TEMPENTITY)
	write_byte(TE_MODEL)
	engfunc(EngFunc_WriteCoord, origin[0])
	engfunc(EngFunc_WriteCoord, origin[1])
	engfunc(EngFunc_WriteCoord, origin[2])
	write_coord(random_num(-100,100))
	write_coord(random_num(-100,100))
	write_coord(random_num(100,200))
	write_angle(random_num(0,360))
	write_short(mdl_gib_head) // Sprite index
	write_byte(0) // bounce
	write_byte(500) // life
	message_end()
	
	// Spine
	message_begin(MSG_BROADCAST,SVC_TEMPENTITY)
	write_byte(TE_MODEL)
	engfunc(EngFunc_WriteCoord, origin[0])
	engfunc(EngFunc_WriteCoord, origin[1])
	engfunc(EngFunc_WriteCoord, origin[2])
	write_coord(random_num(-100,100))
	write_coord(random_num(-100,100))
	write_coord(random_num(100,200))
	write_angle(random_num(0,360))
	write_short(mdl_gib_spine)
	write_byte(0) // bounce
	write_byte(500) // life
	message_end()
	
	// Lung
	for(new i = 0; i < random_num(1,2); i++) 
	{
		message_begin(MSG_BROADCAST,SVC_TEMPENTITY)
		write_byte(TE_MODEL)
		engfunc(EngFunc_WriteCoord, origin[0])
		engfunc(EngFunc_WriteCoord, origin[1])
		engfunc(EngFunc_WriteCoord, origin[2])
		write_coord(random_num(-100,100))
		write_coord(random_num(-100,100))
		write_coord(random_num(100,200))
		write_angle(random_num(0,360))
		write_short(mdl_gib_lung)
		write_byte(0) // bounce
		write_byte(500) // life
		message_end()
	}
	
	// Parts, 10 times
	for(new i = 0; i < 10; i++) 
	{
		message_begin(MSG_BROADCAST,SVC_TEMPENTITY)
		write_byte(TE_MODEL)
		engfunc(EngFunc_WriteCoord, origin[0])
		engfunc(EngFunc_WriteCoord, origin[1])
		engfunc(EngFunc_WriteCoord, origin[2])
		write_coord(random_num(-100,100))
		write_coord(random_num(-100,100))
		write_coord(random_num(100,200))
		write_angle(random_num(0,360))
		write_short(mdl_gib_flesh)
		write_byte(0) // bounce
		write_byte(500) // life
		message_end()
	}
	
	// Blood
	for(new i = 0; i < 3; i++) 
	{
		new x,y,z
		x = random_num(-100,100)
		y = random_num(-100,100)
		z = random_num(0,100)
		for(new j = 0; j < 3; j++)
		{
			message_begin(MSG_BROADCAST,SVC_TEMPENTITY)
			write_byte(TE_BLOODSPRITE)
			engfunc(EngFunc_WriteCoord, origin[0]+(x*j))
			engfunc(EngFunc_WriteCoord, origin[1]+(y*j))
			engfunc(EngFunc_WriteCoord, origin[2]+(z*j))
			write_short(blood_spray)
			write_short(blood_drop)
			write_byte(229) // color index
			write_byte(15) // size
			message_end()
		}
	}
}
Last edited by YamiKaitou; 06-20-2014 at 18:06.
leandro4422 is offline	
