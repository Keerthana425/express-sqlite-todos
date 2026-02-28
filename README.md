const express = require('express')
const app = express()
app.use(express.json())
const path = require('path')
const dbpath = path.join(__dirname, 'todoApplication.db')
const {open} = require('sqlite')
const sqlite3 = require('sqlite3')
let db = null
const format = require('date-fns/format')
const isValid = require('date-fns/isValid')
const toDate = require('date-fns/toDate')

const initalconnectionServerToDb = async () => {
  try {
    db = await open({
      filename: dbpath,
      driver: sqlite3.Database,
    })
    app.listen(3000, () => {
      console.log('Server Running at port...3000')
    })
  } catch (e) {
    console.log(`DB Error : ${e.message}`)
    process.exit(1)
  }
}

initalconnectionServerToDb()

const checkRequestsQueries = async (request, response, next) => {
  const {search_q, category, priority, status, date} = request.query
  const {todoId} = request.params
  if (category !== undefined) {
    const categoryArray = ['WORK', 'HOME', 'LEARNING']
    const categoryIsInArray = categoryArray.includes(category)
    if (categoryIsInArray === true) {
      request.category = category
    } else {
      response.status(400)
      response.send('Invalid Todo Category')
      return
    }
  }

  if (priority !== undefined) {
    const priorityArray = ['HIGH', 'MEDIUM', 'LOW']
    const priorityIsInArray = priorityArray.includes(priority)
    if (priorityIsInArray === true) {
      request.priority = priority
    } else {
      response.status(400)
      response.send('Invalid Todo Priority')
      return
    }
  }

  if (status !== undefined) {
    const statusArray = ['TO DO', 'IN PROGRESS', 'DONE']
    const statusIsInArray = statusArray.includes(status)
    if (statusIsInArray === true) {
      request.status = status
    } else {
      response.status(400)
      response.send('Invalid Todo Status')
      return
    }
  }

  if (date !== undefined) {
    try {
      const myDate = new Date(date)

      const formatedDate = format(new Date(date), 'yyyy-MM-dd')

      const result = toDate(
        new Date(
          `${myDate.getFullYear()}-${
            myDate.getMonth() + 1
          }-${myDate.getDate()}`,
        ),
      )

      const isValidDate = await isValid(result)

      if (isValidDate === true) {
        request.date = formatedDate
      } else {
        response.status(400)
        response.send('Invalid Due Date')
        return
      }
    } catch (e) {
      response.status(400)
      response.send('Invalid Due Date')
      return
    }
  }

  request.todoId = todoId
  request.search_q = search_q

  next()
}

const checkRequestsBody = (request, response, next) => {
  const {id, todo, category, priority, status, dueDate} = request.body
  const {todoId} = request.params

  if (category !== undefined) {
    categoryArray = ['WORK', 'HOME', 'LEARNING']
    categoryIsInArray = categoryArray.includes(category)

    if (categoryIsInArray === true) {
      request.category = category
    } else {
      response.status(400)
      response.send('Invalid Todo Category')
      return
    }
  }

  if (priority !== undefined) {
    priorityArray = ['HIGH', 'MEDIUM', 'LOW']
    priorityIsInArray = priorityArray.includes(priority)
    if (priorityIsInArray === true) {
      request.priority = priority
    } else {
      response.status(400)
      response.send('Invalid Todo Priority')
      return
    }
  }

  if (status !== undefined) {
    statusArray = ['TO DO', 'IN PROGRESS', 'DONE']
    statusIsInArray = statusArray.includes(status)
    if (statusIsInArray === true) {
      request.status = status
    } else {
      response.status(400)
      response.send('Invalid Todo Status')
      return
    }
  }

  if (dueDate !== undefined) {
    try {
      const myDate = new Date(dueDate)
      const formatedDate = format(new Date(dueDate), 'yyyy-MM-dd')

      const result = toDate(new Date(formatedDate))
      const isValidDate = isValid(result)

      if (isValidDate === true) {
        request.dueDate = formatedDate
      } else {
        response.status(400)
        response.send('Invalid Due Date')
        return
      }
    } catch (e) {
      response.status(400)
      response.send('Invalid Due Date')
      return
    }
  }
  request.todo = todo
  request.id = id

  request.todoId = todoId

  next()
}

app.get(`/todos/`, checkRequestsQueries, async (request, response) => {
  const {search_q = '', priority, status, category} = request
  const catandstatus = obj => {
    return obj.status !== undefined && obj.category !== undefined
  }
  const todoStatus = obj => {
    return obj.status !== undefined
  }
  const todoPriority = obj => {
    return obj.priority !== undefined
  }
  const todoBoth = obj => {
    return obj.status !== undefined && obj.priority !== undefined
  }
  const todocategory = obj => {
    return obj.category !== undefined
  }

  const catandpro = obj => {
    return obj.category !== undefined && obj.priority !== undefined
  }
  let data = null
  let getTodoQuery = ''
  switch (true) {
    case catandstatus(request.query):
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE status='${status}' AND category='${category}';`
      break
    case todocategory(request.query):
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE todo LIKE '%${search_q}%' AND category='${category}'`
      break

    case todoBoth(request.query):
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE todo LIKE '%${search_q}%' AND status='${status}' AND priority='${priority}'`
      break
    case todoStatus(request.query):
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE todo LIKE '%${search_q}%' AND status='${status}'`
      break
    case todoPriority(request.query):
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE todo LIKE '%${search_q}%' AND priority='${priority}'`
      break

    case catandpro(request.query):
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE todo LIKE '%${search_q}%' AND priority='${priority}' AND category='${category}'`
      break
    default:
      getTodoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE todo LIKE '%${search_q}%'`
  }
  data = await db.all(getTodoQuery)
  response.send(data)
})

app.get(`/todos/:todoId/`, checkRequestsQueries, async (request, response) => {
  const {todoId} = request.params
  const todoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE id=${todoId};`
  const dbResponse = await db.get(todoQuery)
  response.send(dbResponse)
})

app.get(`/agenda/`, checkRequestsQueries, async (request, response) => {
  const {date} = request

  const todoQuery = `SELECT id,todo,priority,status,category,due_date as dueDate FROM todo WHERE dueDate='${date}';`
  const dbResponse = await db.all(todoQuery)

  response.send(dbResponse)
})

app.post(`/todos/`, checkRequestsBody, async (request, response) => {
  const {id, todo, priority, status, category, dueDate} = request
  const updateQuerys = `INSERT INTO todo(id, todo, priority, status, category,due_date) VALUES(${id},'${todo}','${priority}','${status}','${category}','${dueDate}');`
  const sd = await db.run(updateQuerys)
  console.log(sd)
  response.send('Todo Successfully Added')
})

app.put(`/todos/:todoId/`, checkRequestsBody, async (request, response) => {
  const {todoId} = request
  const {status, category, priority, todo, dueDate} = request
  let updateQuery = null
  switch (true) {
    case status !== undefined:
      updateQuery = `UPDATE todo SET status='${status}' WHERE id=${todoId};`
      await db.run(updateQuery)
      response.send('Status Updated')
      break
    case category !== undefined:
      updateQuery = `UPDATE todo SET category='${category}' WHERE id=${todoId};`
      await db.run(updateQuery)
      response.send('Category Updated')
      break
    case priority !== undefined:
      updateQuery = `UPDATE todo SET priority='${priority}' WHERE id=${todoId};`
      await db.run(updateQuery)
      response.send('Priority Updated')
      break
    case todo !== undefined:
      updateQuery = `UPDATE todo SET todo='${todo}' WHERE id=${todoId};`
      await db.run(updateQuery)
      response.send('Todo Updated')
      break
    case dueDate !== undefined:
      updateQuery = `UPDATE todo SET due_date='${dueDate}' WHERE id=${todoId};`
      await db.run(updateQuery)
      response.send('Due Date Updated')
      break
  }
})
app.delete(
  `/todos/:todoId/`,
  checkRequestsQueries,
  async (request, response) => {
    const {todoId} = request.params
    const deleteQuery = `DELETE FROM todo WHERE id=${todoId};`
    await db.run(deleteQuery)
    response.send('Todo Deleted')
  },
)
module.exports = app
