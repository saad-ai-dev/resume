# 16. SEQUELIZE ORM

## 16.1 What is Sequelize?
Sequelize is a **Promise-based Node.js ORM** for SQL databases (PostgreSQL, MySQL, SQLite, MSSQL). You used it at Obenan with 100+ models and 400+ migrations.

## 16.2 Core Concepts

### Model Definition
```javascript
const { DataTypes } = require('sequelize');

module.exports = (sequelize) => {
  const User = sequelize.define('User', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    name: {
      type: DataTypes.STRING(100),
      allowNull: false,
      validate: { len: [2, 100] },
    },
    email: {
      type: DataTypes.STRING,
      unique: true,
      validate: { isEmail: true },
    },
    role: {
      type: DataTypes.ENUM('user', 'admin', 'superadmin'),
      defaultValue: 'user',
    },
    metadata: {
      type: DataTypes.JSONB, // PostgreSQL
    },
  }, {
    tableName: 'users',
    timestamps: true,      // createdAt, updatedAt
    paranoid: true,        // soft delete (deletedAt)
    indexes: [
      { fields: ['email'], unique: true },
      { fields: ['role'] },
    ],
  });

  return User;
};
```

### Associations
```javascript
// One-to-Many
User.hasMany(Post, { foreignKey: 'userId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'userId', as: 'author' });

// Many-to-Many
User.belongsToMany(Role, { through: 'user_roles' });
Role.belongsToMany(User, { through: 'user_roles' });

// One-to-One
User.hasOne(Profile, { foreignKey: 'userId' });
Profile.belongsTo(User, { foreignKey: 'userId' });
```

### Queries
```javascript
// Find with associations (eager loading)
const user = await User.findOne({
  where: { id: userId },
  include: [
    { model: Post, as: 'posts', where: { status: 'published' }, required: false },
    { model: Profile, as: 'profile' }
  ],
  attributes: ['id', 'name', 'email'],
});

// Pagination
const { count, rows } = await User.findAndCountAll({
  where: { role: 'admin' },
  limit: 10,
  offset: (page - 1) * 10,
  order: [['createdAt', 'DESC']],
});

// Raw query (when ORM is too limiting)
const results = await sequelize.query(
  'SELECT * FROM users WHERE age > :age',
  { replacements: { age: 25 }, type: QueryTypes.SELECT }
);

// Transactions
const t = await sequelize.transaction();
try {
  await User.create({ name: 'Saad' }, { transaction: t });
  await Profile.create({ userId: user.id }, { transaction: t });
  await t.commit();
} catch (error) {
  await t.rollback();
}
```

### Migrations
```javascript
// npx sequelize migration:generate --name add-users-table
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('users', {
      id: { type: Sequelize.UUID, primaryKey: true },
      name: { type: Sequelize.STRING, allowNull: false },
      email: { type: Sequelize.STRING, unique: true },
      createdAt: { type: Sequelize.DATE },
      updatedAt: { type: Sequelize.DATE },
    });
    await queryInterface.addIndex('users', ['email']);
  },
  down: async (queryInterface) => {
    await queryInterface.dropTable('users');
  },
};
```

### Multi-Tenant Scoping (Your Obenan Experience)
```javascript
// Scope queries by company, user, and location
const reviews = await Review.findAll({
  where: {
    companyId: req.user.companyId,    // Tenant isolation
    locationId: req.params.locationId, // Location scoping
  },
  include: [{ model: Location, where: { companyId: req.user.companyId } }],
});
```

