# jdbcrepo

CREATE OR REPLACE FUNCTION getminimumbalance(
	acc integer,
	mon integer,
	yearr integer)
    RETURNS numeric
    LANGUAGE 'plpgsql'
AS $$
declare
finalMinBal account_p.currbal%type;
changingBal numeric(10,2);
initd Date = make_date(yearr,mon,10);
initbal account_p.currbal%type;
transCur cursor for select * from public.transactions_p where accno = acc
					and (extract(month from tr_date) = mon and extract(year from tr_date) = yearr and extract(day from tr_date)>=10);
eachTransRec transactions_p%rowtype;
begin
open transCur;
initbal = public.getbalancebydate2(acc,initd);
-- raise notice 'initial: %',initbal;
changingBal = initbal;
finalMinBal = initbal;
loop
	
	fetch transCur into eachTransRec;
	exit when not found;
	raise notice '% , % , % , %',eachTransRec.accno,eachTransRec.tr_date,eachTransRec.tr_type,eachTransRec.tr_amt;
	if eachTransRec.tr_type = 'd' then 
		changingBal = changingBal+eachTransRec.tr_amt;
	elsif eachTransRec.tr_type = 'w' then
		changingBal = changingBal-eachTransRec.tr_amt;
	end if;
	
	if changingBal<finalMinBal then
		finalMinBal = changingBal;
		
	end if;
	
end loop;
close transCur;
	
	return finalMinBal;

end; 
$$;


select getminimumbalance(1002,01,2024)






-- CREATE OR REPLACE FUNCTION public.getbalancebydate2(
-- 	acc integer,
-- 	idate date)
--     RETURNS numeric
--     LANGUAGE 'plpgsql'
-- AS $$
-- declare
-- 	totalDeps numeric(10,2) = 0;
-- 	totalWithdraws numeric(10,2) = 0;
-- 	currAmt numeric(10,2)=0;
-- begin
-- 	totalDeps := (select sum(tr_amt) from public.transactions_p where accno = acc and tr_type = 'd' and tr_date between idate and current_date);
-- 	totalWithdraws := (select sum(tr_amt) from public.transactions_p where accno = acc and tr_type = 'w' and tr_date between idate and current_date);
-- 	currAmt := (select currbal from public.account_p where accno = acc);
-- 	raise notice ' % , % , % ',totalDeps,totalWithdraws,currAmt;
-- 	return currAmt+coalesce(totalWithdraws,0)-coalesce(totalDeps,0);
-- end; 
-- $$;

-- select getbalancebydate2(1002,'2023-12-26')
