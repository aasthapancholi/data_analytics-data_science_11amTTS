# Module 2: Working with Databases - Session 3: Insert, Delete, and Update

---

## 1. Create a table called `Playlist` with columns: `id` (INT, primary key), `song_name` (VARCHAR), `artist` (VARCHAR), and `duration` (INT, seconds). Insert a single row for your current favorite song.

### SOLUTION:
* **Tool Used:** XAMPP (phpMyAdmin / MySQL)
* We first create the table `Playlist` with `id` designated as the `PRIMARY KEY`.
* Next, we insert a single record for a favorite song, intentionally including a slight typo in the artist's name (`'Arjit Singh'`) to be resolved in Question 3.

#### SQL Query:
```sql
-- 1. Create Playlist Table
CREATE TABLE Playlist (
    id INT PRIMARY KEY,
    song_name VARCHAR(100) NOT NULL,
    artist VARCHAR(100) NOT NULL,
    duration INT NOT NULL
);

-- 2. Insert Favorite Song
INSERT INTO Playlist (id, song_name, artist, duration) 
VALUES (1, 'Kesariya', 'Arjit Singh', 268);
```

#### Table State after Task 1:
| id | song_name | artist | duration (seconds) |
|:---|:---|:---|:---|
| 1 | Kesariya | Arjit Singh | 268 |

---

## 2. Insert 3 new rows into the `Playlist` table for songs you recently listened to on Spotify, including their `song_name`, `artist`, and `duration`.

### SOLUTION:
We add 3 additional songs into the `Playlist` table using a multi-row `INSERT INTO` statement:
- One track with duration under 120 seconds (to test Task 4).
- Tracks by `'AP Dhillon'` with durations above and below 180 seconds (to test Task 5).

#### SQL Query:
```sql
INSERT INTO Playlist (id, song_name, artist, duration) VALUES
(2, 'Brown Munde', 'AP Dhillon', 265),
(3, 'Summer High', 'AP Dhillon', 178),
(4, 'Morning Interlude', 'Oliver Tree', 110);
```

#### Table State after Task 2:
| id | song_name | artist | duration (seconds) | Description |
|:---|:---|:---|:---|:---|
| 1 | Kesariya | Arjit Singh | 268 | From Task 1 (contains typo) |
| 2 | Brown Munde | AP Dhillon | 265 | AP Dhillon (> 180s) |
| 3 | Summer High | AP Dhillon | 178 | AP Dhillon (<= 180s) |
| 4 | Morning Interlude | Oliver Tree | 110 | Short track (< 120s) |

---

## 3. Update the artist name for one of your Playlist entries to fix a typo (for example, change 'Arjit Singh' to 'Arijit Singh') using the UPDATE statement with a WHERE clause.

### SOLUTION:
To correct the typo in the artist's name for song ID 1, we use the `UPDATE` statement with a `WHERE` clause targeting `id = 1`.

#### SQL Query:
```sql
UPDATE Playlist
SET artist = 'Arijit Singh'
WHERE id = 1;
```

#### Table State after Task 3:
| id | song_name | artist | duration (seconds) | Status |
|:---|:---|:---|:---|:---|
| 1 | Kesariya | **Arijit Singh** | 268 | **Updated (Typo Fixed)** |
| 2 | Brown Munde | AP Dhillon | 265 | Unchanged |
| 3 | Summer High | AP Dhillon | 178 | Unchanged |
| 4 | Morning Interlude | Oliver Tree | 110 | Unchanged |

---

## 4. Delete a song from the `Playlist` table where the duration is less than 120 seconds using the DELETE statement and a WHERE clause.
*(Hint: Make sure your WHERE clause is specific so you don’t accidentally delete all rows.)*

### SOLUTION:
To remove any short song (under 120 seconds), we execute `DELETE FROM` with the filter `WHERE duration < 120`.

#### SQL Query:
```sql
DELETE FROM Playlist
WHERE duration < 120;
```

#### Table State after Task 4:
| id | song_name | artist | duration (seconds) |
|:---|:---|:---|:---|
| 1 | Kesariya | Arijit Singh | 268 |
| 2 | Brown Munde | AP Dhillon | 265 |
| 3 | Summer High | AP Dhillon | 178 |

*(Row `id = 4` (`'Morning Interlude'`, 110s) was deleted as `110 < 120`).*

---

## 5. Write an SQL statement that would update the song_name for all songs by 'AP Dhillon' in your Playlist to add '(Remix)' at the end of the name, but only if the duration is more than 180 seconds.
*(Constraint: Combine UPDATE with WHERE to target only the correct rows.)*

### SOLUTION:
We use the `CONCAT()` function in MySQL to append `' (Remix)'` to the existing `song_name`. The `WHERE` clause combines two conditions using `AND`:
1. `artist = 'AP Dhillon'`
2. `duration > 180`

#### SQL Query:
```sql
UPDATE Playlist
SET song_name = CONCAT(song_name, ' (Remix)')
WHERE artist = 'AP Dhillon' 
  AND duration > 180;
```

#### Verification Query:
```sql
SELECT * FROM Playlist;
```

#### Final Output:
| id | song_name | artist | duration (seconds) | Note |
|:---|:---|:---|:---|:---|
| 1 | Kesariya | Arijit Singh | 268 | Artist not AP Dhillon (Not modified) |
| 2 | **Brown Munde (Remix)** | AP Dhillon | 265 | **Updated** (AP Dhillon AND 265 > 180) |
| 3 | Summer High | AP Dhillon | 178 | Not updated (178 is not > 180) |

---

### Key Takeaways for DML (Data Manipulation Language):
- **`INSERT`**: Adds new rows to a table.
- **`UPDATE`**: Modifies existing data. Always use a specific `WHERE` clause to avoid updating all rows.
- **`DELETE`**: Removes existing data. Always use a specific `WHERE` clause to avoid clearing the entire table.
- **String Manipulation (`CONCAT`)**: Useful in `SET` clauses when dynamically modifying existing text values.
