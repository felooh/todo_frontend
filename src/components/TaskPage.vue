<template>
    <div class="container">
        <div class="add_task">
            <form @submit.prevent="submitForm">
                <div class="form-group">
                    <label for="title">Title</label>
                    <input type="text" class="form-control" id="title" v-model="title">
                </div>
                <div class="form-group">
                    <label for="description">Description</label>
                    <textarea class="form-control" id="description" v-model="description"></textarea>
                </div>
                <br>
                <div class="form-group">
                    <button class="btn btn-primary" type="submit" >Add Task</button>
                </div>
            </form>
        </div>
    </div>
    <div class="container">
        <div>
            <div class="tasks_content">
                <h1>Tasks</h1>
                <ul class="tasks_list">
                    <li v-for="task in tasks" :key="task.id">
                        <h2>{{ task.title }}</h2>
                        <p>{{ task.description }}</p>
                        <button class="btn btn-success"  @click="toggleTask(task)">
                            {{ task.completed ? 'Undo' : 'Complete' }}
                        </button>
                        <button  class="btn btn-danger" @click="deleteTask(task)">Delete</button>
                    </li>
                </ul>
            </div>
        </div>
    </div>
    
   
</template>

<script>
import axios from "axios";
    export default {
        data() {
            return {
                // tasks
                tasks: [''],
                title: '',
                description: ''
            }
        },
        methods: {
            async getData() {
                try {
                    // fetch tasks
                    const response = await axios.get('http://localhost:8000/api/tasks/');
                    // set the data returned as tasks
                    this.tasks = response.data; 
                } catch (error) {
                    // log the error
                    console.log(error);
                }
            },
        
            created() {
                // Fetch tasks on page load
                this.getData();
        },
            async submitForm(){
                    try {
                        // Send a POST request to the API
                        const response = await axios.post('http://localhost:8000/api/tasks/', {
                            title: this.title,
                            description: this.description,
                            completed: false
                        });
                        // Append the returned data to the tasks array
                        this.tasks.push(response.data);
                        // Reset the title and description field values.
                        this.title = '';
                        this.description = '';
                    } catch (error) {
                        // Log the error
                        console.log(error);
                }
        },
            async toggleTask(task){
                    try{

                        // Send a request to API to update the task
                        const response = await axios.put(`http://localhost:8000/api/tasks/${task.id}/`, {
                            completed: !task.completed,
                            title: task.title,
                            description: task.description
                        });

                        // Get the index of the task being updated
                        let taskIndex = this.tasks.findIndex(t => t.id === task.id);

                        // Reset the tasks array with the new data of the updated task

                        this.tasks = this.tasks.map((task) => {
                            if(this.tasks.findIndex(t => t.id === task.id) === taskIndex){
                                return response.data;
                            }
                            return task;
                        });

                    }catch(error){

                        // Log any error
                        console.log(error);
                    }
              },
              async deleteTask(task){

                    // Confirm if one wants to delete the task
                    let confirmation = confirm("Do you want to delete this task?"); 

                    if(confirmation){
                        try{

                        // Send a request to delete the task
                        await axios.delete(`http://localhost:8000/api/tasks/${task.id}`);

                        // Refresh the tasks
                        this.getData();
                        }catch(error){

                        // Log any error

                        console.log(error)
                        }
                    }
                }
            }
        }
</script>