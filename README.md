clc
clear
pma=2850;
load('statemat.mat');
pw=[.862;.9;0.878;0.834;0.88;0.841;0.832;.806;0.74;0.737;0.715;0.727;0.704;0.75;0.721;0.8;0.754;0.837;0.87;0.88;0.856;0.811;0.9;0.887;0.896;0.861;0.755;0.816;0.801;0.88;0.722;0.776;0.80;0.729;0.726;0.705;0.78;0.695;0.724;0.724;0.743;0.744;0.80;0.881;0.885;0.909;0.94;0.89;0.942;0.97;1;0.952];    
pd=[0.93;1;0.98;0.96;0.94;0.77;0.75];   
phw=[0.67;0.63;0.60;0.59;0.59;0.6;0.74;0.86;0.95;0.96;0.96;0.95;0.95;0.95;0.93;0.94;0.99;1;1;0.96;0.91;0.83;0.73;0.63];   
php=[0.63;0.62;0.60;0.58;0.59;0.65;0.72;0.85;0.95;0.99;1;0.99;0.93;0.92;0.90;0.88;0.90;0.92;0.96;0.98;0.96;0.90;0.80;0.70];
phs=[0.64;0.60;0.58;0.56;0.56;0.58;0.64;0.76;0.87;0.95;0.99;1;0.99;1;1;0.97;0.96;0.96;0.93;0.92;0.92;0.93;0.87;0.72];
loady=[];
for i=1:length(pw)
    loadw=[];
   for j=1:length(pd)
    if i>=9 && i<18
        loadh=php;
    elseif i>=18 && i<31
        loadh=phs;
    elseif i>=31 && i<44
        loadh=php;
    else
        loadh=phw;
    end
    loadh=loadh*pd(j);
    loadw=[loadw;loadh];
   end
   loadw=loadw*pw(i);
   loady=[loady;loadw];
end
loady=loady*pma;
loadplot=loady;
figure
plot(1:length(loady),loady) 
title('Chronological load curve over year')
xlabel('Time(hours)');
ylabel('Load power (MW)');

pmax=max(loady);
lg=loady==pmax;
i=1;
pdr(i,1)=sum(lg);
pdr(i,2)=pmax;
while(pmax>0)
    loady=loady.*(~(lg));
    i=i+1;
    pmax=max(loady);
    lg=loady==pmax;
pdr(i,1)=sum(lg);
pdr(i,2)=pmax;
end
pdr(end,:)=[];
t(1)=pdr(1,1);
for i=2:length(pdr(:,1))
    t(i)=t(i-1)+pdr(i,1);
end
figure
plot(t,pdr(:,2))
title('Load duration curve over year')
xlabel('Time(hours)');
ylabel('Load power (MW)');
figure
plot(1:24,phw)
title('Chronological load curve for winter day')
xlabel('Time(hours)');
ylabel('Load power % of daily peak');
figure
plot(1:24,phs)
title('Chronological load curve for summer day')
xlabel('Time(hours)');
ylabel('Load power % of daily peak');
figure
plot(1:24,php)
title('Chronological load curve for spring day')
xlabel('Time(hours)');
ylabel('Load power % of daily peak');


figure
plot(1:24*7,loadplot((4*7*24)+1:(5*7*24)))
title('Chronological load curve for winter week(5)')
xlabel('Time(hours)');
ylabel('Load power (MW)');
figure
plot(1:24*7,loadplot((24*7*24)+1:(25*7*24)))
title('Chronological load curve for summer week(25)')
xlabel('Time(hours)');
ylabel('Load power (MW)');
figure
plot(1:24*7,loadplot((34*7*24)+1:(35*7*24)))
title('Chronological load curve for spring week(35)')
xlabel('Time(hours)');
ylabel('Load power (MW)');


figure
[tw, ppw]=loadduration(phw);
plot(tw,ppw)
title('Load duration curve for winter day')
xlabel('Time(hours)');
ylabel('Load power % of daily peak');
figure
[ts, pps]=loadduration(phs);
plot(ts,pps)
title('Load duration curve for summer day')
xlabel('Time(hours)');
ylabel('Load power % of daily peak');
figure
[tp, ppp]=loadduration(php);
plot(tp,ppp)
title('Load duration curve for spring day')
xlabel('Time(hours)');
ylabel('Load power % of daily peak');



figure
[tw5, pp5]=loadduration(loadplot((4*7*24)+1:(5*7*24)));
plot(tw5,pp5)
title('Load duration curve for winter week(5)')
xlabel('Time(hours)');
ylabel('Load power (MW)');
figure
[tw25, pp25]=loadduration(loadplot((24*7*24)+1:(25*7*24)));
plot(tw25,pp25)
title('Load duration curve for summer week(25)')
xlabel('Time(hours)');
ylabel('Load power (MW)');
figure
[tw35, pp35]=loadduration(loadplot((34*7*24)+1:(35*7*24)));
plot(tw35,pp35)
title('Load duration curve for spring week(35)')
xlabel('Time(hours)');
ylabel('Load power (MW)');




gs=statemat;
gr=[12;20;50;76;100;155;197;350;400];
gn=[5;4;6;4;3;4;3;1;2];
for i=1:length(gs(1,:))
    gst(1:9,i)=gs(:,i).*gr;
gst(10,i)=sum(gst(1:9,i));
end

op=[0.02;0.1;0.01;0.02;0.04;0.04;0.05;0.08;0.12];
for i=1:length(gs(1,:))
    prd=1;
    for j=1:9
         prd=nchoosek(gn(j),gs(j,i))*prd;
        prd=op(j)^(gs(j,i))*prd*(1-op(j))^(gn(j)-gs(j,i));
    end 
    gst(11,i)=prd;
end

tl=gst(10,:);
pr=gst(11,:);
gn=[];
for ggg=1:length(tl)
[migst minpp]=min(tl);
  gn(ggg,:)=[ migst pr(minpp)];
  tl(minpp)=[];pr(minpp)=[];
  
end
ggn=gn(1,:);
for gh=2:length(gst(10,:))
if gn(gh-1,1)==gn(gh,1)
    ggn(end,:)=[ggn(end,1) ggn(end,2)+gn(gh,2)];
else
   ggn=[ggn;gn(gh,1) gn(gh,2)]; 
end
end

cggn=ggn;
for cgh=1:length(ggn(:,1))-1
cggn(end-cgh,2)=sum(ggn(end-cgh:end,2));
end

figure
bar(ggn(:,1),ggn(:,2))
title('Individual probability function')
xlabel('Capacity outage');
ylabel('f_y (y)');
figure
bar(cggn(:,1),cggn(:,2))
title('Cumulative probability function')
xlabel('Capacity outage');
ylabel('F_y (y)');
tblg(1)=ggn(1,1);
tblp(1)=ggn(1,2);
for kk=1:35
    lgG=(cggn(2:end,1)>=(kk-1)*100 & cggn(2:end,1)<(kk)*100);
    tblg(kk+1)=mean(nonzeros(lgG.*ggn(2:end,1)));
    tblp(kk+1)=sum(nonzeros(lgG.*ggn(2:end,2)));
end
tblv=tblp;
for cgh2=1:length(tblp(:,1))-1
tblv(end-cgh2)=sum(tblp(end-cgh2:end));
end
figure
bar(tblg,tblp)
title('Individual probability function(36 group)')
xlabel('Capacity outage');
ylabel('f_y (y)');
figure
bar(tblg,tblv)
title('Cumulative probability function(36 group)')
xlabel('Capacity outage');
ylabel('F_y (y)');
%for day
[lopled lopld]=estimlopl(loadplot(1:24),cggn);
%for year
[lopley loply]=estimlopl(loadplot,cggn);

function[t, pp]=loadduration(loady)
pmax=max(loady);
lg=loady==pmax;
i=1;
pdr(i,1)=sum(lg);
pdr(i,2)=pmax;
while(pmax>0)
    loady=loady.*(~(lg));
    i=i+1;
    pmax=max(loady);
    lg=loady==pmax;
pdr(i,1)=sum(lg);
pdr(i,2)=pmax;
end
pdr(end,:)=[];
t(1)=pdr(1,1);
for i=2:length(pdr(:,1))
    t(i)=t(i-1)+pdr(i,1);
end
pp=pdr(:,2);
end

function[Lople Lopl]=estimlopl(loadplot,cggn)
L=3405-loadplot;
F=cggn(:,2);
for ik=1:length(L)
    Lopl(ik)=interp1(cggn(:,1), F,L(ik));
end
Lople=sum(Lopl);
end
![image](https://github.com/janax28/Matlab-Code/assets/143566132/3d417151-dec3-4aac-80f7-a42f4d2bf218)

