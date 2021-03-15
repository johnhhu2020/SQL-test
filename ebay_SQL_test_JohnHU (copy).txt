drop table if exists p_cac_t_ebay_lstg;

create table p_cac_t_ebay_lstg (
item_id int,
listing_date date,
price float);

insert into p_cac_t_ebay_lstg
values
(1,'2018-01-01',5),
(2,'2018-01-01',3),
(3,'2018-01-01',4),
(4,'2018-01-01',5),
(2,'2018-03-01',2),
(3,'2018-07-01',1),
(2,'2018-04-01',2),
(3,'2018-08-01',2);

---Question 1 Get the latest listing_date when it had the lowest price in history;

SELECT item_id, MAX(listing_date),price FROM p_cac_t_ebay_lstg WHERE (item_id,price) IN
(SELECT c.item_id,MIN(c.price) FROM p_cac_t_ebay_lstg c  GROUP BY c.item_id)
GROUP BY item_id,price ;

-----------------------------------------------------------------------------------------


drop table if exists p_cac_t_ebay_trans;

create table p_cac_t_ebay_trans(
trans_id int,
item_id int,
buyer_id int,
seller_id int,
trans_timestamp timestamp,
quantity int);

insert into p_cac_t_ebay_trans
values
(1,1,5,110,'2018-01-04 22:00:01',2),
(2,2,7,121,'2018-01-01 10:00:01',3),
(3,2,8,121,'2018-01-01 16:00:01',4),
(4,4,8,113,'2019-01-04 22:00:01',5),
(5,1,7,114,'2019-01-04 22:00:01',6),
(6,2,5,115,'2019-01-04 20:00:01',2),
(7,3,11,116,'2019-01-04 22:00:01',3),
(8,4,12,117,'2019-01-04 22:00:01',4),
(9,3,13,118,'2018-07-04 22:00:01',5),
(9,3,12,119,'2019-01-04 22:00:01',5);

---Question 2 find the top 2 buyers who bought from more than 1 seller ranked by GMV
(price * quantity) and their total GMV:

SELECT t.buyer_id,MAX(l.price*t.quantity) AS GMV  FROM p_cac_t_ebay_trans t LEFT JOIN p_cac_t_ebay_lstg l
ON t.item_id=l.item_id
GROUP BY t.buyer_id HAVING COUNT(t.seller_id)>=2
ORDER BY GMV DESC LIMIT 2;

-----------------------------------------------------------------------------------------

drop table if exists p_cac_t_trans_fdback;

create table p_cac_t_trans_fdback (
trans_id int,
feedback_timestamp timestamp,
seller_id int,
buyer_id int,
feedback_score int);

insert into p_cac_t_trans_fdback
values
(1,'2018-01-04 22:30:01',110,5,1),
(2,'2018-01-01 11:00:01',121,7,0),
(3,'2018-01-02 01:00:01',121,8,1),
(4,'2018-01-04 22:30:01',110,5,1),
(5,'2018-01-04 22:30:01',110,5,-1),
(6,'2018-01-04 22:30:01',110,5,1);

---Question 3: Count all items positive, neutral, and negative feedbacks. (1 = positve, 0 =
neutral, -1 = negative):

SELECT t.item_id,f.feedback_score,COUNT(f.feedback_score) FROM p_cac_t_ebay_trans t LEFT JOIN p_cac_t_trans_fdback f
    ON t.trans_id=f.trans_id
GROUP BY t.item_id,f.feedback_score ORDER BY t.item_id;

SELECT f.feedback_score,COUNT(t.item_id) FROM p_cac_t_trans_fdback f LEFT JOIN p_cac_t_ebay_trans t
ON t.trans_id=f.trans_id GROUP BY f.feedback_score;

---------------------------------------------------------------------------------

---No new table is needed:

---Question 4: Calculate % of transactions from sellers who have 50%+ positive feedback
ratio

SELECT  (SELECT COUNT(feedback_score) FROM p_cac_t_trans_fdback WHERE feedback_score>0)/
                  (SELECT COUNT(feedback_score) FROM p_cac_t_trans_fdback) AS percentage FROM p_cac_t_trans_fdback GROUP BY percentage;
