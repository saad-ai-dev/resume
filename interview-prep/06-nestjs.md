# 6. NESTJS

## 6.1 What is NestJS?
NestJS is an **opinionated, Angular-inspired Node.js framework** built with TypeScript. It uses decorators, dependency injection, and a modular architecture.

## 6.2 Core Architecture

```
┌─────────────────────────────────────┐
│              Module                   │
│  ┌─────────┐ ┌──────────┐ ┌──────┐ │
│  │Controller│ │  Service  │ │Guard │ │
│  │(Routes)  │→│(Business  │ │(Auth)│ │
│  │          │ │ Logic)    │ │      │ │
│  └─────────┘ └──────────┘ └──────┘ │
│              ┌──────────┐           │
│              │   Pipe    │           │
│              │(Validate) │           │
│              └──────────┘           │
└─────────────────────────────────────┘
```

### Module
```typescript
@Module({
  imports: [DatabaseModule, AuthModule],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Make available to other modules
})
export class UsersModule {}
```

### Controller
```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {} // DI

  @Get()
  findAll(@Query('page') page: number) {
    return this.usersService.findAll(page);
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  @UseGuards(AuthGuard)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

### Service
```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepo: Repository<User>,
  ) {}

  async findAll(page: number): Promise<User[]> {
    return this.usersRepo.find({
      skip: (page - 1) * 10,
      take: 10,
    });
  }

  async findOne(id: number): Promise<User> {
    const user = await this.usersRepo.findOne({ where: { id } });
    if (!user) throw new NotFoundException(`User #${id} not found`);
    return user;
  }
}
```

### Guards (Authentication)
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    if (!token) throw new UnauthorizedException();

    try {
      const payload = jwt.verify(token, SECRET);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }
}
```

### Pipes (Validation)
```typescript
// DTO with class-validator
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsStrongPassword()
  password: string;
}

// NestJS automatically validates using the DTO
@Post()
@UsePipes(new ValidationPipe({ whitelist: true }))
create(@Body() dto: CreateUserDto) { }
```

