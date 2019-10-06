# rgbzero

--constants
clr={8,1,2,11,10,12,7}
clrl={14,13,13,10,7,6,6}
idx={2,3,0,0,0,0,7,1,0,5,4,6}

coin_pass={}
diff=0
coin_c={}
s={}

true_rnd=rnd(0xffff.ffff)

p_up_flinch={}
for i=0,13 do
 p_up_flinch[i]={}
end

for i=1,7 do
 coin_c[i]={}
	for j=1,5 do
	 coin_c[i][j]=sget(i-1,95+j)
	end
end

--init
function _init()
 music()
 coins=0
 dt=0
 camx=0
 ptc={}
 c=
 {
  x=24,--874,--790,--24,
  y=44,
  xs=0,
  ys=0,
  g=.1,
  c=0,
  f=false,
  sprt=0
 }
 b🅾️=0
 b❎=0
 boss_c=7--ceil(rnd(7))
end

--init functs
function shoot(x,y,t,clr)
 local ang=atan2(c.x+4-x,c.y+4-y)
 --normal
 if t==1 then
  new_shot(x,y,1,clr,ang,1.3)
	 b.stand_max=(90+rnd(15))*((90-diff)/100)
 --triple
 elseif t==2 then
  for i=-.1,.1,.1 do
   new_shot(x,y,1,clr,ang+i,.8)
  end
	 b.stand_max=(100+rnd(15))*((90-diff)/100)
 --circle
 elseif t==3 then
  for i=0,1,1/12 do
   new_shot(x,y,1,clr,ang+i,.5)
  end
	 b.stand_max=(120+rnd(15))*((90-diff)/100)
 --rain
 elseif t==4 then
  new_shot(x,y,2,clr,.25,1.6)
	 b.stand_max=(120+rnd(15))*((90-diff)/100)
 --homing
 elseif t==5 then
  new_shot(x,y,3,clr,rnd(),.4)
	 b.stand_max=(120+rnd(15))*((90-diff)/100)
 end
end

function new_shot(x,y,t,clr,ang,spd)
 b.stand_t=0
 return add(s,{
 x=x,
 y=y,
 dt=0,
 tailx={},
 taily={},
 xs=cos(ang)*spd,
 ys=sin(ang)*spd,
 t=t,
 c=clr
 })
end

--update
function _update60()
 dt+=1
 
 if btn(❎) then
  b❎+=1
 else
  b❎=0
 end
 if btn(🅾️) then
  b🅾️+=1
 else
  b🅾️=0
 end
 
 c.xs,c.w=0
 if c.invinc and c.invinc>60 then
  c.xs=c.f and .4 or -.4
 else
	 if btn(⬅️) and not btn(➡️) then
	  c.xs=-.7
	  c.f=true
	  c.sprt+=.2
	  c.w=true
	 elseif btn(➡️) and not btn(⬅️) then
		 c.xs=.7
		 c.f=false
	  c.sprt+=.2
	  c.w=true
	 end
 end
 
 c.sprt=c.sprt%2
 
 c.x+=c.xs
 c.x=(mid(0,c.x,128*8-8))
 --xcoll
 for i=flr(c.x/8),ceil(c.x/8) do
  for j=flr(c.y/8),ceil(c.y/8) do
	  --spr(mget(i,j),i*8,j*8)
	  j=j%14
	  if mget(i,j)<8 then
		  if band(mget(i,j),c.c)>0 then
				 if (c.xs>0) c.x=i*8-8
				 if (c.xs<0) c.x=i*8+8
			 end
		 end
	 end
 end
 
 c.ys=min(c.ys+c.g,4)
 
 if (c.j and (b🅾️==1 or b❎==1))c.ys=-2.2 sfx(15)
 c.j=nil
 c.y+=c.ys
 --if (c.y>128) c.y-=128
 c.y=c.y%112
 
 --ycoll
 for i=flr(c.x/8),ceil(c.x/8) do
  for j=flr(c.y/8),ceil(c.y/8) do
	  --spr(mget(i,j),i*8,j*8)
	  j=j%14
	  if mget(i,j)<8 then
		  if band(mget(i,j),c.c)>0 then
				 if c.ys>0 then
				  if (c.ys>1) sfx(16)
				  c.y=j*8-8
				  c.ys=0
				  c.j=true
				 end
				 if (c.ys<0) c.y=j*8+8 c.ys=0
			 end
		 end
	 end
 end
 
 --p up
 for i=flr(c.x/8),ceil(c.x/8) do
  for j=flr(c.y/8),ceil(c.y/8) do
	  j=j%14
   local tile_c=mget(i,j)
   local old_c=c.c
   if abs(i*8-c.x)<5 and abs(j*8-c.y)<5 and
      tile_c>16 and tile_c<24 then
    c.c=bor(c.c,tile_c-16)
    if old_c~=c.c then
     sfx(20)
     p_up_flinch[j][i]=40
    end
   elseif tile_c>24 and tile_c<32 then
    c.c=band(c.c,bxor(tile_c-24,7))
    if old_c~=c.c then
     sfx(21)
     p_up_flinch[j][i]=40
    end
   elseif tile_c>40 and tile_c<48 then
    c.c=tile_c-40
    if old_c~=c.c then
     sfx(20)
     p_up_flinch[j][i]=40
    end
   end
  end
 end
 
 
 --coins
 for i=flr(c.x/8),ceil(c.x/8) do
  for j=flr(c.y/8),ceil(c.y/8) do
	  --spr(mget(i,j),i*8,j*8)
	  j=j%14
	  local tile_typ=mget(i,j)
	  if abs(i*8-c.x)<5 and abs(j*8-c.y)<5 and
	     tile_typ>8 and tile_typ<16 then
		  local touch_c=band(tile_typ-8,c.c)
		  if touch_c>0 then
		   local c_count=0
     if (band(touch_c,1)==1) coins+=1 c_count+=1
     if (band(touch_c,2)==2) coins+=1 c_count+=1
     if (band(touch_c,4)==4) coins+=1 c_count+=1
     sfx(16+c_count)
     create_ptc(i*8+4,j*8+4,touch_c,20)
     mset(i,j,bxor(tile_typ,touch_c))
			 end
		 end
	 end
 end
  
 --enter boss
 if c.x>930 and not in_boss then
  in_boss=true
  music(6)
  for i=111,112 do
   for j=1,3 do
    mset(i,j,7)
   end
  end
  b=
  {
   x=1070,
   y=-16,
   tx=980,
   ty=16,
   stand_max=30,
   stand_t=0,
   f=true,
   hp=0,
   dt=0
  }
 end
 
 srand(true_rnd)
 true_rnd=rnd(0xffff.ffff)
 --boss behavior
 if b then
  if (filled_hp) b.stand_t+=1
  b.dt+=1
  b.x=b.x*.97+b.tx*.03
  b.y=b.y*.97+b.ty*.03
  if abs(b.x-b.tx)<1 and
  abs(b.y-b.ty)<1
  or b.stand_t>b.stand_max then
   if not filled_hp then
		  if #coin_pass+b.hp<coins then
		   if (dt%6==0)sfx(40)add(coin_pass,{x=106,y=0,c=ceil(rnd(7))})
		  else
		   filled_hp=true
		   b.stand_t=999
		  end
   elseif b.dt>500 then
	   --if b.stand_t>b.stand_max then
	    if diff<10 then
	     if diff%5<4 then
	      shoot(b.x+8,b.y+8,1,boss_c)
	     else
	      shoot(b.x+8,b.y+8,2,boss_c)
      end
	    elseif diff<20 then
	     if diff%5<2 then
	      shoot(b.x+8,b.y+8,1,boss_c)
	     elseif diff%5<4 then
	      shoot(b.x+8,b.y+8,2,boss_c)
	     else
	      shoot(b.x+8,b.y+8,3,boss_c)
      end
	    elseif diff<30 then
	     if diff%5<1 then
	      shoot(b.x+8,b.y+8,3,boss_c)
	     elseif diff%5<2 then
	      shoot(b.x+8,b.y+8,4,boss_c)
	     else
	      shoot(b.x+8,b.y+8,1,boss_c)
      end
	    elseif diff<40 then
	     if diff%5<1 then
	      shoot(b.x+8,b.y+8,4,boss_c)
	     elseif diff%5<2 then
	      shoot(b.x+8,b.y+8,5,boss_c)
	     elseif diff%5<3 then
	      shoot(b.x+8,b.y+8,3,boss_c)
	     elseif diff%5<4 then
	      shoot(b.x+8,b.y+8,5,boss_c)
	     else
	      shoot(b.x+8,b.y+8,2,boss_c)
      end
	    elseif diff<50 then
	     shoot(b.x+8,b.y+8,ceil(rnd(4))+1,boss_c)
	    elseif diff<90 then
	     shoot(b.x+8,b.y+8,0,boss_c)
	     b.stand_max=40-(diff/2)
	    elseif not ending then
	     ending=0
	    end
	    if not ending then
		    sfx(39)
		    create_ptc(b.x+8,b.y+8,boss_c,20)
		    b.stand_t=0
		    b.tx=906+sin(rnd())*41+41
		    b.ty=rnd(30)
		    boss_c=ceil(rnd(6))
		    b.hp-=1
		    diff+=1
	    end
	   --end
   end
  end
  
  b.f=b.tx>947
 end
 if ending then
  ending+=1
  if ending==100 then
   b.ty=-40
   music(-1,2000)
   sfx(42)
  end
	end
 
 foreach(ptc,move_ptc)
 foreach(coin_pass,move_coin_pass)
 foreach(s,move_shot)
 
 if c.invinc then
  c.invinc-=1
  if (c.invinc<=0) c.invinc=nil
 end
 
 if in_boss then
  if (camx<128*7-4) camx+=1
 else
  camx=mid(0,c.x-64,128*7-4) 
 end
 camera(camx,-16)
end

--update functs
function create_ptc(x,y,c,n)
 for i=1,n do
  local a=rnd()
	 add(ptc,{
		 x=x,
		 y=y,
		 c=c,
		 xs=sin(a)*(3+rnd()),
		 ys=cos(a)*(3+rnd()),
		 dt=0,
		 tailx={},
		 taily={},
		 dtmx=30+rnd(15)
	 })
 end
end

function move_coin_pass(p)
 p.x=p.x*.9+(39+b.hp)*.1
 if abs(p.x-(39+b.hp))<2 then
  b.hp+=1
  del(coin_pass,p)
 end
end

function move_shot(p)
 if p.wait then
  p.wait-=1
  if (p.wait<=0) p.wait=nil
 else
  p.dt+=1
	 p.tailx[#p.tailx+1]=p.x+sin(p.dt/24)
	 p.taily[#p.taily+1]=p.y+cos(p.dt/24)
	 --homing
	 if p.t==3 then
	  if p.dt<60 then
	   p.ang=atan2(c.x+4-p.x,c.y+4-p.y)
	  end
	  p.xs+=cos(p.ang)*.04
	  p.ys+=sin(p.ang)*.04
	 end
	 p.x+=p.xs
	 p.y+=p.ys
	-- p.xs=p.xs*.85
	-- p.ys=p.ys*.85
	 --collision
	 if not c.invinc and
	  band(p.c,c.c)>0 and
	  abs(p.x-c.x-4)<4 and
	  abs(p.y-c.y-4)<4 then
	  c.invinc=90
	  coins-=1
		 create_ptc(c.x+4,c.y+4,band(p.c,c.c),20)
		 sfx(41)
	 end
	-- if p.x>camx-20 or p.x<camx-148 or
	-- p.y>140 or p.y<0 then
	--  del(ptc,p)
	-- end
	 if p.t==2 then
	  if p.y<-20 then
	   for i=0+rnd(12)+camx,128+camx,12 do
	    local sht=new_shot(i,-16,1,p.c,.75,1)
	    sht.wait=rnd(90)
	   end
	   del(s,p)
	  end
	 else
		 if p.x>camx+148 or p.x<camx-20 or
		 p.y>148 or p.y<-20 then
		  del(s,p)
		 end
	 end
 end
end

function move_ptc(p)
 p.dt+=1
 p.tailx[#p.tailx+1]=p.x
 p.taily[#p.taily+1]=p.y
 p.x+=p.xs
 p.y+=p.ys
 p.xs=p.xs*.85
 p.ys=p.ys*.85
 if (p.dt>p.dtmx) del(ptc,p)
end

--draw
function _draw()
 cls(15)
 
 first_tile_x=flr(camx/8)
 --soft tiles
 p_ups={}
 cns={}
 for i=first_tile_x,first_tile_x+16 do
	 for j=-1,13 do
	  tile_typ=mget(i,j%14)
	  if tile_typ>16 then
	   add(p_ups,{x=i,y=j,c=tile_typ})
	  elseif tile_typ>8 then
	   add(cns,{x=i,y=j,c=tile_typ-8})
	  else
	   spr(tile_typ,i*8,j*8)
	  end
	 end
 end
 pal()
 for jj=-112,112,112 do
	 --black char
	 if c.c==0 then
	  pal(1,0)spr(35,c.x,c.y+jj,1,1,c.f)pal()
	 --clr char
	 else
	  if not c.invinc or c.invinc%4<2 then
			 for i=0,7 do
			  for j=0,7 do
			   if sget(i+(not c.j and 24 or c.w and flr(c.sprt)*8+8 or 0),16+j)==1 then
			   pset(c.x+(c.f and 7-i or i),c.y+j+jj,
			   clr[bor(idx[pget(c.x+(c.f and 7-i or i),c.y+j)],c.c)]
			   )
			   end
			  end
			 end
		 end
		end
	end
	--solid tiles
 for i=first_tile_x,first_tile_x+16 do
	 for j=-1,13 do
	  local tile_typ=mget(i,j%14)
	  if (tile_typ<16 and band(c.c,tile_typ)>0) spr(tile_typ+48,i*8,j*8)
	 end
 end
 
 --boss
 if in_boss then
  draw_boss(b.x,b.y,f)
 end
 
 foreach(cns,draw_coin)
 pal()
 foreach(p_ups,draw_p_up)
 pal()
 --ptcs
 foreach(s,draw_shot)
 pal()
 foreach(ptc,draw_ptc)
 --screen bounds
 camera(0,0)
	rectfill(0,0,128,10,0)
	rectfill(0,124,128,127,0)
 --boss hp
 if b and b.hp>0 then
  rectfill(42,3,42+b.hp,5,6)
  rect(42,2,43+50,6,7)
 end
 --colors ui
 circfill(4,4,3,clr[c.c])
 circfill(12,4,2,clr[band(c.c,1)])
 circfill(20,4,2,clr[band(c.c,2)])
 circfill(28,4,2,clr[band(c.c,4)])
 circ(4,4,3,6)
 circ(12,4,2,6)
 circ(20,4,2,6)
 circ(28,4,2,6)
 pset(3,3,7)
 pset(11,3,7)
 pset(19,3,7)
 pset(27,3,7)
 
 --coins ui
	if not c.invinc or c.invinc%4<2 then
	 print(coins,116,1,7)
	 for i=1,5 do
	  pal(i,coin_c[7][i])
	 end
	 spr(211+t()*8%6,106,0)
	 pal()
 end
 foreach(coin_pass,draw_coin_pass)
 pal()
 
 if ending then
	 if ending>200 then
	  rectfill(20,30,107,80,0)
	  print("congratulations!",32,40,clr[ceil(dt/3)%6+1])
	 end
	 if ending>300 then
	  print("final score: "..coins,33,60,clr[ceil((dt-2)/3)%6+1])
	 end
 end
 
 --final colors
 pal(15,0x80,1)--black bg
 pal(1,0x8c,1)--blue
 --if (dt%2==0) pal(12,11,1)--teal
 --pal(2,8+dt%2*0x84,1)--purple
 pal(5,1,1)
 
 --?stat(0),48,122,8
 --?ending,68,122,8
end

--draw functs
function draw_ptc(p)
 if (p.dt%4<2)line(p.x,p.y,p.tailx[max(1,#p.tailx-3)],p.taily[max(1,#p.taily-3)],p.dt<8 and 7 or clr[p.c])
end

function draw_p_up(p)
 function dr()
	 srand(p.x+p.y)
	 local r=rnd()
	 local xx,yy=p.x*8+sin(t()/3+r),p.y*8+cos(t()/2+r)
	 spr(p.c,xx,yy)
	 sspr(7,103,12,12,xx-1,yy-1) 
 end
 local flch=p_up_flinch[p.y%14][p.x]
 if flch and flch>0 then
  p_up_flinch[p.y%14][p.x]-=1
  if (p_up_flinch[p.y%14][p.x]%8<4) dr()
  if (p_up_flinch[p.y%14][p.x]<=0) p_up_flinch[p.y][p.x]=nil
 else
  dr()
 end
end

function draw_coin(p)
 for i=1,5 do
  pal(i,coin_c[p.c][i])
 end
 spr(211+t()*8%6,p.x*8,p.y*8)
end

function draw_coin_pass(p)
 if dt%2==0 then
	 for i=1,5 do
	  pal(i,coin_c[p.c][i])
	 end
	 spr(211+t()*8%6,p.x,p.y)
 end
end

function draw_shot(p)
-- for i=1,5 do
--  pal(i,coin_c[p.c][i])
-- end
 local r,sclr=6,clr[p.c]==7 and 13 or clr[p.c]
 for i=#p.tailx,max(#p.tailx-10),-1 do
  circfill(p.tailx[i],p.taily[i],r,sclr)
  r-=.4
 end
 circfill(p.x,p.y,5,sclr)
 r,sclr=4,clrl[p.c]
 for i=#p.tailx,max(#p.tailx-10),-1 do
  circfill(p.tailx[i],p.taily[i],r,sclr)
  r-=.25
 end
 local r=2
 for i=#p.tailx,max(#p.tailx-10),-1 do
  circfill(p.tailx[i],p.taily[i],r,7)
  r-=.2
 end
end

function draw_boss(x,y,f)
 for i=1,5 do
  pal(i,coin_c[boss_c][i])
 end
 spr(218,x,y,2,2,b.f)
 pal()
end

--todo
--[[

opening screen?
ending?
]]
