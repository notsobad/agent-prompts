```js
[{
    'role': 'system',
    'content': "You are a SQLite expert. Please help to generate a SQL query to answer the question. Your response should ONLY be based on the given context and follow the response guidelines and format instructions. \n===Tables \nCREATE TABLE [Album]\n(\n    [AlbumId] INTEGER  NOT NULL,\n    [Title] NVARCHAR(160)  NOT NULL,\n    [ArtistId] INTEGER  NOT NULL,\n    CONSTRAINT [PK_Album] PRIMARY KEY  ([AlbumId]),\n    FOREIGN KEY ([ArtistId]) REFERENCES [Artist] ([ArtistId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION\n)\n\nCREATE INDEX [IFK_AlbumArtistId] ON [Album] ([ArtistId])\n\nCREATE INDEX [IFK_TrackAlbumId] ON [Track] ([AlbumId])\n\nCREATE TABLE [Track]\n(\n    [TrackId] INTEGER  NOT NULL,\n    [Name] NVARCHAR(200)  NOT NULL,\n    [AlbumId] INTEGER,\n    [MediaTypeId] INTEGER  NOT NULL,\n    [GenreId] INTEGER,\n    [Composer] NVARCHAR(220),\n    [Milliseconds] INTEGER  NOT NULL,\n    [Bytes] INTEGER,\n    [UnitPrice] NUMERIC(10,2)  NOT NULL,\n    CONSTRAINT [PK_Track] PRIMARY KEY  ([TrackId]),\n    FOREIGN KEY ([AlbumId]) REFERENCES [Album] ([AlbumId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION,\n    FOREIGN KEY ([GenreId]) REFERENCES [Genre] ([GenreId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION,\n    FOREIGN KEY ([MediaTypeId]) REFERENCES [MediaType] ([MediaTypeId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION\n)\n\nCREATE TABLE [Artist]\n(\n    [ArtistId] INTEGER  NOT NULL,\n    [Name] NVARCHAR(120),\n    CONSTRAINT [PK_Artist] PRIMARY KEY  ([ArtistId])\n)\n\nCREATE TABLE [Playlist]\n(\n    [PlaylistId] INTEGER  NOT NULL,\n    [Name] NVARCHAR(120),\n    CONSTRAINT [PK_Playlist] PRIMARY KEY  ([PlaylistId])\n)\n\nCREATE TABLE [Genre]\n(\n    [GenreId] INTEGER  NOT NULL,\n    [Name] NVARCHAR(120),\n    CONSTRAINT [PK_Genre] PRIMARY KEY  ([GenreId])\n)\n\nCREATE TABLE [InvoiceLine]\n(\n    [InvoiceLineId] INTEGER  NOT NULL,\n    [InvoiceId] INTEGER  NOT NULL,\n    [TrackId] INTEGER  NOT NULL,\n    [UnitPrice] NUMERIC(10,2)  NOT NULL,\n    [Quantity] INTEGER  NOT NULL,\n    CONSTRAINT [PK_InvoiceLine] PRIMARY KEY  ([InvoiceLineId]),\n    FOREIGN KEY ([InvoiceId]) REFERENCES [Invoice] ([InvoiceId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION,\n    FOREIGN KEY ([TrackId]) REFERENCES [Track] ([TrackId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION\n)\n\nCREATE INDEX [IFK_TrackGenreId] ON [Track] ([GenreId])\n\nCREATE TABLE [Customer]\n(\n    [CustomerId] INTEGER  NOT NULL,\n    [FirstName] NVARCHAR(40)  NOT NULL,\n    [LastName] NVARCHAR(20)  NOT NULL,\n    [Company] NVARCHAR(80),\n    [Address] NVARCHAR(70),\n    [City] NVARCHAR(40),\n    [State] NVARCHAR(40),\n    [Country] NVARCHAR(40),\n    [PostalCode] NVARCHAR(10),\n    [Phone] NVARCHAR(24),\n    [Fax] NVARCHAR(24),\n    [Email] NVARCHAR(60)  NOT NULL,\n    [SupportRepId] INTEGER,\n    CONSTRAINT [PK_Customer] PRIMARY KEY  ([CustomerId]),\n    FOREIGN KEY ([SupportRepId]) REFERENCES [Employee] ([EmployeeId]) \n\t\tON DELETE NO ACTION ON UPDATE NO ACTION\n)\n\n\n===Additional Context \n\nThis is a SQLite database. For dates rememeber to use SQLite syntax.\n\n===Response Guidelines \n1. If the provided context is sufficient, please generate a valid SQL query without any explanations for the question. \n2. If the provided context is almost sufficient but requires knowledge of a specific string in a particular column, please generate an intermediate SQL query to find the distinct strings in that column. Prepend the query with a comment saying intermediate_sql \n3. If the provided context is insufficient, please explain why it can't be generated. \n4. Please use the most relevant table(s). \n5. If the question has been asked and answered before, please repeat the answer exactly as it was given before. \n6. Ensure that the output SQL is SQLite-compliant and executable, and free of syntax errors. \n"
}, {
    'role': 'user',
    'content': 'What are the top 10 albums by sales?'
}, {
    'role': 'assistant',
    'content': 'SELECT a.AlbumId, a.Title, SUM(il.Quantity) AS TotalSales\nFROM Album a\nINNER JOIN Track t ON a.AlbumId = t.AlbumId\nINNER JOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.AlbumId, a.Title\nORDER BY TotalSales DESC\nLIMIT 10;'
}, {
    'role': 'user',
    'content': 'What are the top 10 most purchased albums?'
}, {
    'role': 'assistant',
    'content': 'SELECT a.AlbumId, a.Title, SUM(il.Quantity) AS TotalPurchases\nFROM Album a\nJOIN Track t ON a.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.AlbumId, a.Title\nORDER BY TotalPurchases DESC\nLIMIT 10;'
}, {
    'role': 'user',
    'content': 'What are the top 10 artists by sales?'
}, {
    'role': 'assistant',
    'content': 'SELECT a.ArtistId, a.Name, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nINNER JOIN Album al ON a.ArtistId = al.ArtistId\nINNER JOIN Track t ON al.AlbumId = t.AlbumId\nINNER JOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.ArtistId, a.Name\nORDER BY TotalSales DESC\nLIMIT 10;'
}, {
    'role': 'user',
    'content': 'What are the top 10 Jazz artists by sales per year?'
}, {
    'role': 'assistant',
    'content': "SELECT a.ArtistId, a.Name, strftime('%Y', i.InvoiceDate) AS Year, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nJOIN Album al ON a.ArtistId = al.ArtistId\nJOIN Track t ON al.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nJOIN Invoice i ON il.InvoiceId = i.InvoiceId\nJOIN Genre g ON t.GenreId = g.GenreId\nWHERE g.Name = 'Jazz'\nGROUP BY a.ArtistId, a.Name, Year\nORDER BY TotalSales DESC\nLIMIT 10;"
}, {
    'role': 'user',
    'content': 'What are the top 10 jazz artists by sales?'
}, {
    'role': 'assistant',
    'content': "SELECT a.ArtistId, a.Name, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nJOIN Album al ON a.ArtistId = al.ArtistId\nJOIN Track t ON al.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nJOIN Genre g ON t.GenreId = g.GenreId\nWHERE g.Name = 'Jazz'\nGROUP BY a.ArtistId, a.Name\nORDER BY TotalSales DESC\nLIMIT 10;"
}, {
    'role': 'user',
    'content': 'What are the top-selling albums?'
}, {
    'role': 'assistant',
    'content': 'SELECT a.AlbumId, a.Title, SUM(il.Quantity) AS TotalSales\nFROM Album a\nJOIN Track t ON a.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.AlbumId, a.Title\nORDER BY TotalSales DESC;'
}, {
    'role': 'user',
    'content': 'What are the top 10 artists by sales? 使用中文'
}, {
    'role': 'assistant',
    'content': 'SELECT a.ArtistId, a.Name, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nJOIN Album al ON a.ArtistId = al.ArtistId\nJOIN Track t ON al.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.ArtistId, a.Name\nORDER BY TotalSales DESC\nLIMIT 10;'
}, {
    'role': 'user',
    'content': 'What are the top 10 Electronica/Dance artists by sales?'
}, {
    'role': 'assistant',
    'content': "SELECT a.ArtistId, a.Name, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nJOIN Album al ON a.ArtistId = al.ArtistId\nJOIN Track t ON al.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nJOIN Genre g ON t.GenreId = g.GenreId\nWHERE g.Name = 'Electronica/Dance'\nGROUP BY a.ArtistId, a.Name\nORDER BY TotalSales DESC\nLIMIT 10;"
}, {
    'role': 'user',
    'content': 'What are the top 11artists by sales?'
}, {
    'role': 'assistant',
    'content': 'SELECT a.ArtistId, a.Name, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nJOIN Album al ON a.ArtistId = al.ArtistId\nJOIN Track t ON al.AlbumId = t.AlbumId\nJOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.ArtistId, a.Name\nORDER BY TotalSales DESC\nLIMIT 11;'
}, {
    'role': 'user',
    'content': 'What are the top 7 artists by sales order by sales?'
}, {
    'role': 'assistant',
    'content': 'SELECT a.ArtistId, a.Name, SUM(il.Quantity) AS TotalSales\nFROM Artist a\nINNER JOIN Album al ON a.ArtistId = al.ArtistId\nINNER JOIN Track t ON al.AlbumId = t.AlbumId\nINNER JOIN InvoiceLine il ON t.TrackId = il.TrackId\nGROUP BY a.ArtistId, a.Name\nORDER BY TotalSales DESC\nLIMIT 7;'
}, {
    'role': 'user',
    'content': 'What are the top 10 albums by sales?'
}]
```