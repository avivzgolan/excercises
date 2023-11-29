# NodeJs

## Exercice: Is there a problem? _(1 points)_

```javascript
// Call web service and return count user, (got is library to call url)
async function getCountUsers() {
  // Added try/catch for error handling
  try {
    // Since url can change, we want to save it in a variable to quickly update it when necessary.
    const url = "https://my-webservice.moveecar.com/users/count";
    // The body needs to be accessed from the response object
    const response = await got.get(url);
    // Assuming the "got" library responds with json, we need to parse it.
    const { total } = JSON.parse(response.body);

    return { total };
  } catch (error) {
    // Handle error
    console.log(error);
    throw new Error("This is a custom error");
  }
}

// Add total from service with 20
async function computeResult() {
  // Again, error handling.
  try {
    // An async function always returns a promise, which must be awaited or caught with a .then().
    const { total } = await getCountUsers();

    return total + 20;
  } catch (error) {
    console.log(error);
    throw new Error("This is a custom error");
  }
}
```

## Exercice: Is there a problem? _(2 points)_

```javascript
// Call web service and return total vehicles, (got is library to call url)
async function getTotalVehicles() {
  // See previous comment regarding the url variable.
  const url = "https://my-webservice.moveecar.com/vehicles/total";
  // Removed unnecessarry "await", since the "got" call will already return a promise.
  return got.get(url);
}

async function getPlurial() {
  /*
    - Added "async" to use async/await syntax.
    - Replaced .then() with "await" to unify syntax.
    - Moved "total" variable definition to be assigned the result of the "getTotalVehicles" promise. syntax is cleaner and more efficient. Also, using const is considered safer than let since we know "total" does not need to be re-assigned a different value.
    - Accessed body from the response, see previous comment.
  */
  const response = await getTotalVehicles();
  const { total } = JSON.parse(response.body);

  // Fixed syntax to be more readable.
  let message = "many";
  if (total <= 0) {
    message = "none";
  } else if (total <= 10) {
    message = "few";
  }
  return message;
}
```

## Exercice: Unit test _(2 points)_

Write unit tests in jest for the function below in typescript

```typescript
import { expect, test } from "@jest/globals";

function getCapitalizeFirstWord(name: string): string {
  if (name == null) {
    throw new Error("Failed to capitalize first word with null");
  }
  if (!name) {
    return name;
  }
  return name
    .split(" ")
    .map((n) =>
      n.length > 1
        ? n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()
        : n
    )
    .join(" ");
}

test('getCapitalizeFirstWord throws "Failed to capitalize first word with null" when called with a null value', () => {
  // Arrange
  // Act
  // Assert
  expect(() => getCapitalizeFirstWord(null)).toThrow(
    "Failed to capitalize first word with null"
  );
});

test("getCapitalizeFirstWord returns the name value unchanged if undefined", () => {
  // Arrange
  const name = undefined;
  // Act
  const response = getCapitalizeFirstWord(name);
  // Assert
  expect(response).toEqual(name);
});

test("getCapitalizeFirstWord returns the name value unchanged if empty", () => {
  // Arrange
  const name = "";
  // Act
  const response = getCapitalizeFirstWord(name);
  // Assert
  expect(response).toEqual(name);
});

test("getCapitalizeFirstWord returns the name capitalized when name value is valid", () => {
  // Arrange
  const name = "jose villaplana";
  const capitalized = "JOSE villaplana";
  // Act
  const response = getCapitalizeFirstWord(name);
  // Assert
  expect(response).toEqual(capitalized);
});
```

# Angular

## Exercice: Is there a problem and improve the code _(5 points)_

```typescript
import { debounceTime, merge, switchMap } from "rxjs/operators";

@Component({
  selector: "app-users",
  template: `
    <input
      type="text"
      [(ngModel)]="query"
      (ngModelChange)="querySubject.next($event)"
    />
    <div *ngFor="let user of users">
      {{ user.email }}
    </div>
  `,
})
export class AppUsers implements OnInit {
  query = "";
  querySubject = new Subject<string>();

  users: { email: string }[] = [];

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    const debounceTime = 600;
    merge(of(this.query), this.querySubject.asObservable())
      .pipe(
        // Added missing debounceTime operator to introduce a delay between user input changes.
        debounceTime(debounceTime),
        // Replaced concatMap with switchMap to handle the switch to a new observable when the input changes.
        switchMap((q) => this.userService.findUsers(q))
      )
      .subscribe({
        next: (res) => (this.users = res),
      });
  }
}
```

## Exercice: Improve performance _(5 points)_

```typescript
import { Component, Input } from "@angular/core";

@Component({
  selector: "app-users",
  template: `
    <div *ngFor="let user of users">
      {{ getCapitalizeFirstWord(user.name) }}
    </div>
  `,
})
export class AppUsers {
  @Input() users: { name: string }[];

  private capitalizedNamesCache = new Map<string, string>();
  // Added memoization to prevent redundant calls to getCapitalizeFirstWord.
  getCapitalizeFirstWord(name: string): string {
    // Check if the result is already memoized
    if (this.capitalizedNamesCache.has(name)) {
      return this.capitalizedNamesCache.get(name)!;
    }

    // Capitalize the first letter of each word
    const capitalized = name
      .split(" ")
      .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
      .join(" ");

    // Memoize the result
    this.capitalizedNamesCache.set(name, capitalized);

    return capitalized;
  }
}
```

## Exercice: Forms _(8 points)_

Complete and modify `AppUserForm` class to use Angular Reactive Forms. Add a button to submit.

The form should return data in this format

```typescript
{
  email: string; // mandatory, must be a email
  name: string; // mandatory, max 128 characters
  birthday?: Date; // Not mandatory, must be less than today
  address: { // mandatory
    zip: number; // mandatory
    city: string; // mandatory, must contains only alpha uppercase and lower and space
  };
}
```

```typescript
import { Component, EventEmitter, Output } from "@angular/core";
import { FormBuilder, FormGroup, Validators } from "@angular/forms";

@Component({
  selector: "app-user-form",
  template: `
    <form [formGroup]="userForm" (ngSubmit)="doSubmit()">
      <input type="text" formControlName="email" placeholder="email" />
      <input type="text" formControlName="name" placeholder="name" />
      <input type="date" formControlName="birthday" placeholder="birthday" />
      <input type="number" formControlName="zip" placeholder="zip" />
      <input type="text" formControlName="city" placeholder="city" />

      <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `,
})
export class AppUserForm {
  @Output()
  event = new EventEmitter<{
    email: string;
    name: string;
    birthday: Date | undefined;
    address: { zip: number; city: string };
  }>();

  userForm: FormGroup;

  constructor(private formBuilder: FormBuilder) {
    this.userForm = this.formBuilder.group({
      email: ["", [Validators.required, Validators.email]],
      name: ["", [Validators.required, Validators.maxLength(128)]],
      birthday: [""],
      zip: ["", [Validators.required, Validators.pattern(/^\d+$/)]],
      city: ["", [Validators.required, Validators.pattern(/^[a-zA-Z\s]+$/)]],
    });
  }

  doSubmit(): void {
    if (this.userForm.valid) {
      const formData = this.userForm.value;
      this.event.emit(formData);
    }
  }
}
```

# CSS & Bootstrap

## Exercice: Card _(5 points)_

![image](uploads/0388377207d10f8732e1d64623a255b6/image.png)

```html
<div class="container">
  <div class="card mx-auto position-relative" style="max-width: 400px;">
    <span
      class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger"
    >
      123+
      <span class="visually-hidden">unread messages</span>
    </span>
    <div class="card-body">
      <h5 class="card-title">Exercise</h5>
      <p class="card-text">
        Redo this card in css if possible using bootstrap 4 or 5
      </p>
      <div class="container">
        <div class="row gx-0">
          <div class="col-8 text-end">
            <div class="btn-group" role="group" aria-label="Basic example">
              <button type="button" class="btn btn-dark">Got it</button>
              <button type="button" class="btn btn-outline-dark">
                I don't know
              </button>
            </div>
          </div>
          <div class="col-4 text-end">
            <div class="dropdown">
              <button
                class="btn btn-secondary dropdown-toggle"
                type="button"
                id="dropdownMenuButton1"
                data-bs-toggle="dropdown"
                aria-expanded="false"
              >
                Options
              </button>
              <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
                <li><a class="dropdown-item" href="#">Action</a></li>
                <li>
                  <a class="dropdown-item" href="#">Another action</a>
                </li>
                <li>
                  <a class="dropdown-item" href="#">Something else here</a>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

# MongoDb

## Exercice: MongoDb request _(3 points)_

MongoDb collection `users` with schema

```typescript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
  }
```

Complete the query, you have a variable that contains a piece of text to search for. Search by exact email, starts with first or last name and only users logged in for 6 months

```typescript
db.collections('users')
.find({
  // Es6 Shorthand for the "email" field, given that we have an "email" variable defined above.
  email,
  last_connection_date: {
    // Date greater than 6 months ago
    $gte: new Date(new Date() - 6 * 30 * 24 * 60 * 60 * 1000),
    // Date smaaller than current date
    $lt: new Date()
  },
  {
    // Project only necessary fields for the response to improve query time
    _id: 0
    first_name: 1
  }
})
.sort({
  // Did not understand the requirement "starts with first or last name". assumed it refers to sort order.
  first_name: 1,
  last_name: 1
});
```

What should be added to the collection so that the query is not slow?

// Indices should be considered on the "email", "first_name" and "last_name" fields respectively.
// Depending on the use case, they could improve or worsen query performance.
// An index on the these fields could improve performance if queries are more often made for a relatively small amount of documents; However, if queries are not selective or if we frequently query the entire collection without filtering based on these fields, the overhead of maintaining the indices could potentially worsen performance.

## Exercice: MongoDb aggregate _(5 points)_

MongoDb collection `users` with schema

```typescript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
  }
```

Complete the aggregation so that it sends user emails by role ({\_id: 'role', users: [email,...]})

```typescript
db.collections("users").aggregate([
  {
    $match: {
      // Es6 Shorthand for the "email" field, given that we have an "email" variable defined above.
      email,
    },
  },
  {
    $project: {
      role: 1,
      email: 1,
    },
  },
  {
    $group: {
      _id: "$role",
      users: {
        $push: "$$ROOT",
      },
    },
  },
  {
    $project: {
      users: {
        email: "$users.email",
      },
    },
  },
]);
```

## Exercice: MongoDb update _(5 points)_

MongoDb collection `users` with schema

```typescript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
    addresses: {
        zip: number;
        city: string;
    }[]:
  }
```

Update document `ObjectId("5cd96d3ed5d3e20029627d4a")`, modify only `last_connection_date` with current date

```typescript
db.collection("users").findByIdAndUpdate(
  "5cd96d3ed5d3e20029627d4a",
  {
    last_connection_date: new Date(),
  },
  // Assuming we want to return the updated document
  { new: true }
);
```

Update document `ObjectId("5cd96d3ed5d3e20029627d4a")`, add a role `admin`

```typescript
db.collection("users").findByIdAndUpdate(
  "5cd96d3ed5d3e20029627d4a",
  {
    // Assuming "roles" is an array of roles
    $push: {
      roles: "admin",
    },
  },
  { new: true }
);
```

Update document `ObjectId("5cd96d3ed5d3e20029627d4a")`, modify addresses with zip `75001` and replace city with `Paris 1`

```typescript
db.collection("users").findOneAndUpdate(
  {
    _id: "5cd96d3ed5d3e20029627d4a",
    "adresses.zip": "75001",
  },
  {
    $set: {
      "addresses.$.city": "Paris 1"
    }
  },
  { new: true }
);
```
