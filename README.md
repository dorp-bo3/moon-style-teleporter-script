# moon-style-teleporter-script


#using scripts\codescripts\struct;
#using scripts\shared\flag_shared;

function init()
{
	//set teleport cooldown time (needed?)
	//level.cooldowntime = 5;

	level.port_trigger = GetEnt( "pad_trig","targetname" );

	level.port_to_structs = GetEntArray( "port_to", "targetname" );
	
	level.porter_trig = GetEnt( "porter_trig","targetname" );
	level.porter_trig TriggerEnable(0);

	level.trig_count = 0;
	
	thread main(); 
}

function main()
{
	level endon( "intermission" ); 
	level endon( "disconnected" ); 

	while(1)
	{
		level.port_trigger waittill( "trigger", player );
		IPrintLnBold("trigger activated");

		players = getplayers(); 
		for( i=0;i<players.size;i++ )
		{
			if( players[i] isTouching(level.porter_trig ) )
			{
				thread porter_trig();
				teleporter_sounds();
			}
		}
		wait(0.05);
	}
}

function teleporter_sounds()
{
	level.port_trigger StopSound( "kino_cooldown" );
	level.port_trigger PlaySound( "kino_top_spark" ); 
	wait(0.5); 
	level.port_trigger PlaySound( "kino_arc_loop" ); 
	level.port_trigger PlaySound( "kino_top_spark" ); 
	wait(0.5); 
	level.port_trigger PlaySound( "kino_top_spark" ); 
	//wait(2);
	wait(1);
}

function porter_trig()
{
	for( e=0;e<5;e++ ) //set e<# where #/2 is amount of seconds before teleporting when on pad
	{
		level.porter_trig TriggerEnable(1);
		players = getplayers();
			for( i=0;i<players.size;i++ )
			{
				if( players[i] isTouching(level.porter_trig) )
				{
					IPrintLnBold("players still on pad");
					level.trig_count ++;

					players[i] SetElectrified(0.5);
				}
				else
				{
					level.port_trigger StopSound( "kino_top_spark" );
					level.port_trigger StopSound( "kino_arc_loop" );
					level.port_trigger PlaySound( "kino_cooldown" );
					IPrintLnBold("players not on pad");
					e += 999;
					level.trig_count = 0;
				}
			}
		level.porter_trig TriggerEnable(0);
		wait(0.5);

		if( level.trig_count == 5 ) //set this # to same one as line 63
		{
			level.trig_count = 0;
			players[i] thread port_to( i );
		}
	}
}

function port_to( player_num )
{
	PlaySoundAtPosition( "kino_beam_fx", self.origin );
	self SetOrigin( level.port_to_structs[player_num].origin ); 
	self SetPlayerAngles( level.port_to_structs[player_num].angles ); 
	IPrintLnBold( "teleport successful" );
}
