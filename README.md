<?php include 'db.php'; ?>
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>ניהול משימות - גרסת לופי העננית</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --bg: #f0f2f5; --card: #ffffff; --text: #1c1e21; --primary: #1877f2; }
        body.dark { --bg: #18191a; --card: #242526; --text: #e4e6eb; --primary: #2d88ff; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background-color: var(--bg); color: var(--text); margin: 0; padding: 20px; transition: 0.3s; }
        .container { max-width: 1200px; margin: auto; }
        header { display: flex; justify-content: space-between; align-items: center; padding: 20px; background: var(--card); border-radius: 15px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); margin-bottom: 20px; }
        .main-grid { display: grid; grid-template-columns: 300px 1fr; gap: 20px; }
        .sidebar { background: var(--card); padding: 20px; border-radius: 15px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); height: fit-content; }
        .add-task-box { background: var(--card); padding: 20px; border-radius: 15px; margin-bottom: 20px; display: flex; flex-direction: column; gap: 10px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        textarea { width: 100%; min-height: 120px; padding: 12px; border: 1px solid #ddd; border-radius: 8px; background: var(--bg); color: var(--text); font-family: inherit; resize: vertical; box-sizing: border-box; font-size: 16px; }
        .btn { padding: 10px 15px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; text-decoration: none; display: inline-flex; align-items: center; justify-content: center; }
        .btn-blue { background: var(--primary); color: white; }
        .board { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; }
        .col { background: rgba(0,0,0,0.05); padding: 15px; border-radius: 12px; min-height: 450px; }
        .task-card { background: var(--card); padding: 15px; border-radius: 10px; margin-bottom: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); border-right: 5px solid #ccc; }
        
        /* עיצוב הטקסט כדי שישמור על המבנה האנכי */
        .task-text { 
            white-space: pre-wrap; 
            word-wrap: break-word; 
            line-height: 1.8; 
            font-size: 15px;
            margin-bottom: 10px;
        }

        .priority-high { border-right-color: #ff4d4d; }
        .priority-medium { border-right-color: #ffd700; }
        .priority-low { border-right-color: #28a745; }
        .actions { display: flex; gap: 10px; justify-content: space-between; border-top: 1px solid rgba(0,0,0,0.05); padding-top: 10px; }
    </style>
</head>
<body class="<?php echo isset($_COOKIE['dark_mode']) ? 'dark' : ''; ?>">

<div class="container">
    <header>
        <h2><i class="fas fa-tasks"></i> ניהול משימות <small style="font-size: 12px; display: block;">הענן של אביב</small></h2>
        <a href="actions.php?action=toggle_dark" class="btn btn-blue"><i class="fas fa-moon"></i> מצב לילה</a>
    </header>

    <div class="main-grid">
        <div class="sidebar">
            <h3><i class="fas fa-lightbulb" style="color: gold;"></i> רעיונות</h3>
            <form action="actions.php?action=add_idea" method="POST">
                <textarea name="text" placeholder="הדבק רעיון מהוואטסאפ..." required></textarea>
                <button type="submit" class="btn btn-blue" style="margin-top:10px; width:100%;">שמור רעיון</button>
            </form>
        </div>

        <div>
            <form class="add-task-box" action="actions.php?action=add_task" method="POST">
                <textarea name="text" placeholder="הדבק כאן את הטקסט מהוואטסאפ..." required></textarea>
                <div style="display:flex; gap:10px;">
                    <select name="priority" style="flex:1;">
                        <option value="low">רגיל 🟢</option>
                        <option value="medium">חשוב 🟡</option>
                        <option value="high">דחוף! 🔴</option>
                    </select>
                    <button type="submit" class="btn btn-blue" style="flex:1;">הוסף למערכת</button>
                </div>
            </form>

            <div class="board">
                <?php
                $cols = ['todo' => 'לביצוע', 'review' => 'בבדיקה', 'done' => 'בוצע'];
                foreach($cols as $status => $title) {
                    echo "<div class='col'><h4>$title</h4>";
                    $tasks = mysqli_query($conn, "SELECT * FROM tasks WHERE status='$status' ORDER BY id DESC");
                    while($t = mysqli_fetch_assoc($tasks)) {
                        // פונקציה חכמה שמוסיפה ירידת שורה לפני כל מספר (למשל לפני 2. או 3.)
                        $formatted_text = preg_replace('/(?<!^)(\d+\.)/', "\n$1", $t['task_text']);
                        
                        echo "<div class='task-card priority-{$t['priority']}'>
                                <div class='task-text'>$formatted_text</div>
                                <div class='actions'>";
                        $next = ($status == 'todo') ? 'review' : (($status == 'review') ? 'done' : '');
                        if($next) echo "<a href='actions.php?action=move&id={$t['id']}&to=$next' class='btn btn-blue' style='font-size:11px;'>קדם</a>";
                        echo "<a href='actions.php?action=delete&id={$t['id']}' style='color:red;'><i class='fas fa-trash'></i></a>
                              </div></div>";
                    }
                    echo "</div>";
                }
                ?>
            </div>
        </div>
    </div>
</div>
</body>
</html>
