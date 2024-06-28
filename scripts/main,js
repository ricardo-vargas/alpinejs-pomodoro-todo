function todoApp() {
    return {
        tasks: [],
        newTask: '',
        estimatedPomodoros: '',
        totalPomodoros: 0,
        DateTime: luxon.DateTime,
        draggingIndex: null,
        activeTaskIndex: null,
        timerSeconds: this.pomodoroLengthInMinutes * 60,
        isTimerRunning: false,
        timerInterval: null,
        timerStatus: 'Work',
        currentCycle: 0,
        pomodoroLengthInMinutes: 25,
        shortBreakLenghtInMinutes: 5,
        longBreakLenghtInMinutes: 15,
        longBreakInterval: 4,


        init() {
            this.loadState();
            this.$watch('tasks', () => this.saveState());
            this.$watch('activeTaskIndex', () => this.saveState());
            this.$watch('timerSeconds', () => this.saveState());
            this.$watch('isTimerRunning', () => this.saveState());
            this.$watch('timerStatus', () => this.saveState());
            this.$watch('currentCycle', () => this.saveState());
        },

        addTask() {
            const pomodoros = parseInt(this.estimatedPomodoros);
            const estimatedMinutes = this.calculateEstimatedMinutes(pomodoros);
            const startTime = this.calculateStartTime();
            const endTime = this.calculateEndTime(startTime, estimatedMinutes);

            this.tasks.push({
                id: Date.now(),
                name: this.newTask,
                estimatedPomodoros: pomodoros,
                completedPomodoros: 0,
                estimatedMinutes,
                startTime: startTime.toFormat('HH:mm'),
                endTime: endTime.toFormat('HH:mm'),
                editing: false,
                editPomodoros: pomodoros,
                isCompleted: false
            });


            this.totalPomodoros += pomodoros;
            this.newTask = '';
            this.estimatedPomodoros = '';

            if (this.activeTaskIndex === null) {
                this.activeTaskIndex = this.tasks.length - 1;
            }

            this.updateAllTimes();

        },

        deleteTask(index) {
            this.totalPomodoros -= this.tasks[index].estimatedPomodoros;
            this.tasks.splice(index, 1);
            if (this.activeTaskIndex === index) {
                if (this.tasks.length > 0) {
                    this.activeTaskIndex = 0;
                }
            } else if (this.activeTaskIndex > index) {
                this.activeTaskIndex--;
            }
            this.updateAllTimes();
        },

        editTask(index) {
            this.tasks[index].editing = true;
            this.tasks[index].editPomodoros = this.tasks[index].estimatedPomodoros;
        },

        saveTask(index) {
            const task = this.tasks[index];
            const newPomodoros = parseInt(task.editPomodoros);
            this.totalPomodoros += newPomodoros - task.estimatedPomodoros;
            task.estimatedPomodoros = newPomodoros;
            task.estimatedMinutes = this.calculateEstimatedMinutes(newPomodoros);
            task.editing = false;
            this.updateAllTimes();
        },

        updateAllTimes() {
            let currentTime = this.DateTime.now().setZone('local');
            this.tasks.filter(task => !task.isCompleted).forEach((task, index) => {
                task.startTime = currentTime.toFormat('HH:mm');
                currentTime = currentTime.plus({ minutes: task.estimatedMinutes });
                task.endTime = currentTime.toFormat('HH:mm');
            });
        },

        calculateEstimatedMinutes(pomodoros) {
            let totalMinutes = pomodoros * this.pomodoroLengthInMinutes;
            totalMinutes += (pomodoros - 1) * this.shortBreakLenghtInMinutes;

            // Calculate long  breaks
            const currentTotalPomodoros = this.totalPomodoros + pomodoros;
            const longBreaks = Math.floor(currentTotalPomodoros / this.longBreakInterval) - Math.floor(this.totalPomodoros / this.longBreakInterval);
            totalMinutes += longBreaks * this.longBreakLenghtInMinutes;

            // Remove the last short break if it's followed by a long  break
            if (currentTotalPomodoros % this.longBreakInterval === 0) {
                totalMinutes -= this.shortBreakLenghtInMinutes;
            }

            return totalMinutes;
        },

        calculateStartTime() {
            if (this.tasks.length === 0) {
                return this.DateTime.now().setZone('local');
            }
            const lastTask = this.tasks[this.tasks.length - 1];
            return this.DateTime.fromFormat(lastTask.endTime, 'HH:mm');
        },

        calculateEndTime(startTime, minutes) {
            return startTime.plus({ minutes });
        },

        get totalTime() {
            const totalMinutes = this.tasks.reduce((sum, task) => sum + task.estimatedMinutes, 0);
            const hours = Math.floor(totalMinutes / 60);
            const minutes = totalMinutes % 60;
            return `${hours} hours and ${minutes} minutes`;
        },

        get estimatedCompletion() {
            if (this.tasks.length === 0) return '';
            const lastTask = this.tasks[this.tasks.length - 1];
            return `Estimated completion time: ${lastTask.endTime}`;
        },

        saveState() {
            localStorage.setItem('pomodoro-state', JSON.stringify({
                tasks: this.tasks,
                totalPomodoros: this.totalPomodoros,
                activeTaskIndex: this.activeTaskIndex,
                timerSeconds: this.timerSeconds,
                isTimerRunning: this.isTimerRunning,
                timerStatus: this.timerStatus,
                currentCycle: this.currentCycle
            }));
        },

        loadState() {
            const savedState = localStorage.getItem('pomodoro-state');
            if (savedState) {
                const state = JSON.parse(savedState);
                this.tasks = state.tasks;
                this.totalPomodoros = state.totalPomodoros;
                this.activeTaskIndex = state.activeTaskIndex;
                this.timerSeconds = state.timerSeconds;
                this.isTimerRunning = state.isTimerRunning;
                this.timerStatus = state.timerStatus;
                this.currentCycle = state.currentCycle;
                if (this.isTimerRunning) {
                    this.startTimer();
                }
                this.updateAllTimes();
            } else {
                this.timerSeconds = this.pomodoroLengthInMinutes * 60;
            }
        },

        dragStart(event, index) {
            this.draggingIndex = index;
            event.dataTransfer.effectAllowed = 'move';
        },

        drop(event, index) {
            event.preventDefault();
            const taskToMove = this.tasks[this.draggingIndex];
            this.tasks.splice(this.draggingIndex, 1);
            this.tasks.splice(index, 0, taskToMove);
            if (this.activeTaskIndex === this.draggingIndex) {
                this.activeTaskIndex = index;
            } else if (this.activeTaskIndex === index) {
                this.activeTaskIndex = this.draggingIndex;
            }
            this.draggingIndex = null;
            this.updateAllTimes();
        },

        selectTask(index) {
            this.activeTaskIndex = index;
        },

        completeTask(index) {
            const task = this.tasks[index];
            task.completedPomodoros = task.estimatedPomodoros;
            task.isCompleted = true;
            this.updateAllTimes();
            if (this.activeTaskIndex === index) {
                //select the closest incomplete task
                if (this.tasks.length > 0) {
                    this.activeTaskIndex = this.tasks.findIndex(t => !t.completedPomodoros);
                }
            }
        },

        startTimer() {
            if (!this.isTimerRunning) {
                this.isTimerRunning = true;
                this.timerInterval = setInterval(() => {
                    if (this.timerSeconds > 0) {
                        this.timerSeconds--;
                    } else {
                        this.completePomodoro();
                    }
                }, 1000);
            }
        },

        pauseTimer() {
            this.isTimerRunning = false;
            clearInterval(this.timerInterval);
        },

        resetTimer() {
            this.pauseTimer();
            switch (this.timerStatus) {
                case 'Work':
                    this.timerSeconds = this.pomodoroLengthInMinutes * 60;
                    break;
                case 'Short Break':
                    this.timerSeconds = this.shortBreakLenghtInMinutes * 60;
                    break;
                case 'Long Break':
                    this.timerSeconds = this.longBreakLenghtInMinutes * 60;
                    break;
            }
        },

        resetCycles() {
            this.currentCycle = 0;
            this.totalPomodoros = 0;
            //save the state
            this.saveState();
        },

        totalExecutedPomodoros() {
            //take the executed pomodoros from the timer
            return this.currentCycle;
        },

        completePomodoro() {
            this.pauseTimer();
            if (this.timerStatus === 'Work') {
                if (this.activeTaskIndex !== null && this.activeTaskIndex >= 0) {
                    this.tasks[this.activeTaskIndex].completedPomodoros++;
                    if (this.tasks[this.activeTaskIndex].completedPomodoros >= this.tasks[this.activeTaskIndex].estimatedPomodoros) {
                        this.activeTaskIndex = null;
                    }
                }
                this.currentCycle++;
                if (this.currentCycle % this.longBreakInterval === 0) {
                    this.timerSeconds = this.longBreakLenghtInMinutes * 60;
                    this.timerStatus = 'Long Break';
                } else {
                    this.timerSeconds = this.shortBreakLenghtInMinutes * 60;
                    this.timerStatus = 'Short Break';
                }
            } else {
                this.timerSeconds = this.pomodoroLengthInMinutes * 60;
                this.timerStatus = 'Work';
            }
            this.startTimer();
        },

        formatTime(seconds) {
            const minutes = Math.floor(seconds / 60);
            const remainingSeconds = seconds % 60;
            return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;

        }
    };
}
