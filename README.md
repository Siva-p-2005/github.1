<?php
// Handle form submission to add a task
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $task = isset($_POST['task']) ? trim($_POST['task']) : '';

    if (!empty($task)) {
        // Save task to a file
        $file = 'tasks.txt';
        file_put_contents($file, $task . PHP_EOL, FILE_APPEND | LOCK_EX);
    }
}

// Load tasks from the file
$tasks = [];
if (file_exists('tasks.txt')) {
    $tasks = file('tasks.txt', FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>To-Do List</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        .container {
            background: #fff;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        h1 {
            margin: 0 0 20px;
            font-size: 24px;
        }

        form {
            display: flex;
            margin-bottom: 20px;
        }

        #taskInput {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 3px;
        }

        button {
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 3px;
            background-color: #28a745;
            color: white;
            cursor: pointer;
        }

        button:hover {
            background-color: #218838;
        }

        ul {
            list-style-type: none;
            padding: 0;
        }

        li {
            padding: 10px;
            border-bottom: 1px solid #ddd;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        li button {
            background-color: #dc3545;
        }

        li button:hover {
            background-color: #c82333;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>To-Do List</h1>
        <form id="taskForm" method="POST">
            <input type="text" id="taskInput" name="task" placeholder="Add a new task..." required>
            <button type="submit">Add Task</button>
        </form>
        <ul id="taskList">
            <?php foreach ($tasks as $task): ?>
                <li>
                    <?php echo htmlspecialchars($task, ENT_QUOTES, 'UTF-8'); ?>
                    <button onclick="deleteTask(this)">Delete</button>
                </li>
            <?php endforeach; ?>
        </ul>
    </div>

    <script>
        function deleteTask(button) {
            const listItem = button.parentElement;
            listItem.remove();

            // Remove the task from the file
            const task = listItem.textContent.replace('Delete', '').trim();

            fetch('index.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: `delete_task=${encodeURIComponent(task)}`,
            });
        }

        document.addEventListener('DOMContentLoaded', () => {
            // Handle task deletion via fetch
            if (window.location.search.includes('delete_task=')) {
                const params = new URLSearchParams(window.location.search);
                const taskToDelete = params.get('delete_task');

                if (taskToDelete) {
                    fetch('index.php', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                        },
                        body: `delete_task=${encodeURIComponent(taskToDelete)}`,
                    }).then(() => {
                        window.location.href = 'index.php'; // Refresh the page
                    });
                }
            }
        });
    </script>
</body>
</html>
