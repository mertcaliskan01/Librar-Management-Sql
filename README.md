# Librar-Management-Sql


1.Müsait durumda olan tüm bilim kurgu kitaplarını listele (başlık, yazar, yıl)
<pre lang="md">
SELECT
	Title,
	Author,
	PublishYear
FROM
	Books
WHERE
	Category = 'Bilim Kurgu'
	AND IsAvailable = TRUE;
</pre>
2. 2020 yılından sonra yayınlanan kitapların toplam sayısını bul
<pre lang="md">
SELECT
	COUNT(*) AS total_books
FROM
	Books
WHERE
	PublishYear > 2020;
</pre>
3.En çok ödünç alınan 3 kitabı listele (kitap adı ve ödünç alınma sayısı)
<pre lang="md">
SELECT
	b.Title AS book_name,
	COUNT(l.Id) AS count
FROM
	Loans l
	JOIN Books b ON l.BookId = b.Id
GROUP BY
	b.Title
ORDER BY
	Sayı DESC
LIMIT
	3;
</pre>
4. Bu ay iade edilmesi gereken kitapları listele (30 gün sonra iade)
<pre lang="md">
SELECT
	b.Title AS book_name
FROM
	Loans l
	JOIN Books b ON l.BookId = b.Id
WHERE
	l.IsReturned = FALSE
	AND l.ReturnDate >= DATE_TRUNC('month', CURRENT_DATE)
	AND l.ReturnDate < DATE_TRUNC('month', CURRENT_DATE) + INTERVAL '1 month';
</pre>
5. Hiç ödünç alınmamış kitapları bul (LEFT JOIN kullanarak)
<pre lang="md">
SELECT
	b.Title AS book_name
FROM
	Books b
	LEFT JOIN Loans l ON b.Id = l.BookId
WHERE
	l.Id IS NULL;
</pre>
6. Her kategoriden en eski kitabı listele (kategori, kitap adı, yıl)
<pre lang="md">
SELECT
	b.Category AS category,
	b.Title AS title,
	b.PublishYear AS publish_year
FROM
	Books b
WHERE
	b.PublishYear = (
		SELECT
			MIN(PublishYear)
		FROM
			Books
		WHERE
			Category = b.Category
	);
</pre>
7.Kitap iade işlemini yapan basit bir stored procedure yazın(Bu prosedür, PostgreSQL 11+ sürümünde çalışır.)
<pre lang="md">
CREATE OR REPLACE PROCEDURE return_book(p_loan_id INT)
LANGUAGE plpgsql
AS $$
DECLARE
    v_book_id INT;
BEGIN
    UPDATE Loans
    SET 
        IsReturned = TRUE,
        ReturnDate = CURRENT_DATE
    WHERE Id = p_loan_id;

    SELECT BookId INTO v_book_id FROM Loans WHERE Id = p_loan_id;

    UPDATE Books
    SET IsAvailable = TRUE
    WHERE Id = v_book_id;
END;
$$;

CALL return_book(10);
</pre>
