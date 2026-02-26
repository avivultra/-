
<?php include 'db.php'; ?>
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ניהול משימות - גרסת לופי העננית</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js"></script>
    <style>
        :root { 
            --bg: #f4f7f6; --card: #ffffff; --text: #2d3436; 
            --primary: #0984e3; --secondary: #636e72;
            --low: #00b894; --medium: #fdcb6e; --high: #d63031;
        }
        body.dark { 
            --bg: #1e272e; --card: #2f3542; --text: #f1f2f6; 
            --primary: #70a1ff; --secondary: #a4b0be;
        }
        body { font-family: 'Assistant', 'Segoe UI', sans-serif; background-color: var(--bg); color: var(--text); margin: 0; padding: 20px; transition: background 0.3s; }
        
        .container { max-width: 1300px; margin: auto; }
        
        header { 
            display: flex; justify-content: space-between; align-items: center; 
            padding: 15px 25px; background: var(--card); border-radius: 20px; 
            box-shadow: 0 4px 15px rgba(0,0,0,0.05); margin-bottom: 30px; 
        }

        .search-bar {
            background: var(--bg); border: none; padding: 10px 20px;
            border-radius: 10px; width: 300px; font-family: inherit;
        }

        .main-grid { display: grid; grid-template-columns: 280px 1fr; gap: 25px; }
        
        .sidebar { 
            background: var(--card); padding: 20px; border-radius: 20px; 
            box-shadow: 0 4px 15px rgba(0,0,0,0.05); height: fit-content;
            position: sticky; top: 20px;
        }

        .add-task-box { 
            background: var(--card); padding: 20px; border-radius: 20px; 
            margin-bottom: 25px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); 
        }

        textarea { 
            width: 100%; min-height: 100px; padding: 15px; border: 2px solid var(--bg); 
            border-radius: 12px; background: var(--bg); color: var(--text); 
            font-family: inherit; resize: none; box-sizing: border-box; font-size: 15px;
            transition: border 0.3s;
        }
        textarea:focus { outline: none; border-color: var(--primary); }

        .btn { 
            padding: 12px 20px; border: none; border-radius: 10px; cursor: pointer; 
            font-weight: 600; display: inline-flex; align-items: center; gap: 8px;
            transition: transform 0.2s, opacity 0.2s;
        }
        .btn:hover { transform: translateY(-2px); opacity: 0.9; }
        .btn-blue { background: var(--primary); color: white; }

        .board { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; }
        
        .col { 
            background: rgba(0,0,0,0.03); padding: 15px; border-radius: 18px; 
            min-height: 600px; border: 1px dashed rgba(0,0,0,0.1);
        }
        .col h4 { margin-top: 0; display: flex; justify-content: space-between; align-items: center; }
        .task-count { font-size: 12px; background: var(--secondary); color: white; padding: 2px 8px; border-radius: 10px; }

        .task-card { 
            background: var(--card); padding: 18px; border-radius: 15px; 
            margin-bottom: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.03); 
            border-right: 6px solid #ccc; cursor: grab; transition: 0.2s;
        }
        .task-card:active { cursor: grabbing; }
        .task-card:hover { box-shadow: 0 8px 20px rgba(0,0,0,0.08); }
        
        .task-text { white-space: pre-wrap; line-height: 1.6; margin-bottom: 12px; }
        
        /* תגיות עדיפות */
        .priority-high { border-right-color: var(--high); }
        .priority-medium { border-right-color: var(--medium); }
        .priority-low { border-right-color: var(--low); }

        .badge { font-size: 10px; padding: 3px 8px; border-radius: 5px; text-transform: uppercase; font-weight: bold; }
        .badge-high { background: #ffeaa7; color: #d63031; }

        .actions { 
            display: flex; gap: 15px; justify-content: flex-end; 
            border-top: 1px solid rgba(0,0,0,0.05); padding-top: 10px; font-size: 14px;
        }
    </style>
</head>
<body class="<?php echo isset($_COOKIE['dark_mode']) ? 'dark' : ''; ?>">

<div class="container">
    <header>
        <div>
            <h2 style="margin:0;"><i class="fas fa-rocket"></i> הענן של אביב</h2>
            <span style="font-size: 12px; color: var(--secondary);">מערכת ניהול משימות לצוות</span>
        </div>
        <input type="text" id="taskSearch" class="search-bar" placeholder="חפש משימה...">
        <a href="actions.php?action=toggle_dark" class="btn btn-blue"><i class="fas fa-moon"></i> מצב לילה</a>
    </header>

    <div class="main-grid">
        <div class="sidebar">
            <h3><i class="fas fa-lightbulb" style="color: gold;"></i> רעיונות מהירים</h3>
            <form action="actions.php?action=add_idea" method="POST">
                <textarea name="text" placeholder="יש רעיון חדש?"></textarea>
                <button type="submit" class="btn btn-blue" style="margin-top:10px; width:100%;">שמור רעיון</button>
            </form>
        </div>

        <div>
            <form class="add-task-box" action="actions.php?action=add_task" method="POST">
                <textarea name="text" placeholder="מה המשימה הבאה שלך? (אפשר להדביק מהוואטסאפ)" required></textarea>
                <div style="display:flex; gap:15px; margin-top:15px;">
                    <select name="priority" style="flex:1; padding:10px; border-radius:10px; border: 2px solid var(--bg);">
                        <option value="low">עדיפות רגילה 🟢</option>
                        <option value="medium">חשוב לביצוע 🟡</option>
                        <option value="high">דחוף ביותר 🔴</option>
                    </select>
                    <button type="submit" class="btn btn-blue" style="flex:1;">הוסף למערכת</button>
                </div>
            </form>

            <div class="board">
                <?php
                $cols = ['todo' => 'לביצוע', 'review' => 'בבדיקה', 'done' => 'בוצע'];
                foreach($cols as $status => $title) {
                    echo "<div class='col' id='col-$status'>
                            <h4>$title <span class='task-count'></span></h4>
                            <div class='task-list' data-status='$status'>";
                    
                    // שימוש ב-Prepared Statement (מומלץ מאוד להוסיף בשלב הבא ב-actions.php)
                    $tasks = mysqli_query($conn, "SELECT * FROM tasks WHERE status='$status' ORDER BY id DESC");
                    while($t = mysqli_fetch_assoc($tasks)) {
                        $safe_text = htmlspecialchars($t['task_text']);
                        $formatted_text = preg_replace('/(?<!^)(\d+\.)/', "\n$1", $safe_text);
                        
                        echo "<div class='task-card priority-{$t['priority']}' data-id='{$t['id']}'>
                                <div class='task-text'>$formatted_text</div>
                                <div class='actions'>
                                    <span style='font-size:10px; color:var(--secondary); margin-left:auto;'>ID: #{$t['id']}</span>
                                    <a href='actions.php?action=delete&id={$t['id']}' style='color:var(--secondary);' onclick='return confirm(\"למחוק?\")'><i class='fas fa-trash'></i></a>
                                </div>
                              </div>";
                    }
                    echo "</div></div>";
                }
                ?>
            </div>
        </div>
    </div>
</div>

<script>
    // פונקציית חיפוש
    document.getElementById('taskSearch').addEventListener('input', function(e) {
        const term = e.target.value.toLowerCase();
        document.querySelectorAll('.task-card').forEach(card => {
            const text = card.innerText.toLowerCase();
            card.style.display = text.includes(term) ? 'block' : 'none';
        });
    });

    // הפעלת גרירה ושחרור
    document.querySelectorAll('.task-list').forEach(list => {
        new Sortable(list, {
            group: 'tasks',
            animation: 150,
            ghostClass: 'ghost',
            onEnd: function(evt) {
                const taskId = evt.item.getAttribute('data-id');
                const newStatus = evt.to.getAttribute('data-status');
                
                // כאן אפשר להוסיף קריאת AJAX כדי לעדכן את המסד בלי רענון דף
                window.location.href = `actions.php?action=move&id=${taskId}&to=${newStatus}`;
            }
        });
    });
</script>
</body>
</html>
