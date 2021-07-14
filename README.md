# FMDBMigrationManager

[![CI Status](https://img.shields.io/travis/qq912276337/FMDBMigrationManager.svg?style=flat)](https://travis-ci.org/qq912276337/FMDBMigrationManager)
[![Version](https://img.shields.io/cocoapods/v/FMDBMigrationManager.svg?style=flat)](https://cocoapods.org/pods/FMDBMigrationManager)
[![License](https://img.shields.io/cocoapods/l/FMDBMigrationManager.svg?style=flat)](https://cocoapods.org/pods/FMDBMigrationManager)
[![Platform](https://img.shields.io/cocoapods/p/FMDBMigrationManager.svg?style=flat)](https://cocoapods.org/pods/FMDBMigrationManager)

## Example

~~To run the example project, clone the repo, and run `pod install` from the Example directory first.~~ 
Example has no sample code.

Example taken from https://programmer.group/ios-uses-fmdb-and-fmdb-migration-manager-to-upgrade-the-database-version.html

Create a new class: Migration follows fmdb Migration protocol, name: upgrade description (can be set to nil), version: version number of current database, array: database operation;

```
#import <Foundation/Foundation.h>
#import "FMDBMigrationManager.h"

@interface Migration : NSObject<FMDBMigrating>

@property (nonatomic, readonly) NSString *name;
@property (nonatomic, readonly)uint64_t version;

/**
 @param name Database upgrade description
 @param version Current version number
 @param updateArray Database operation, since there may be more than one, use array
 @return  Migration
 */
- (instancetype)initWithName:(NSString *)name andVersion:(uint64_t)version andExecuteUpdateArray:(NSArray *)updateArray;

- (BOOL)migrateDatabase:(FMDatabase *)database error:(out NSError *__autoreleasing *)error;

@end
```
```
#import "Migration.h"

@interface Migration()

@property (nonatomic, copy) NSString *myName;
@property (nonatomic, assign) uint64_t myVersion;
@property (nonatomic, strong) NSArray *updateArray;

@end

@implementation Migration

- (instancetype)initWithName:(NSString *)name andVersion:(uint64_t)version andExecuteUpdateArray:(NSArray *)updateArray
{
    if (self = [super init]) {
        _myName = name;
        _myVersion = version;
        _updateArray = updateArray;
    }
    return self;
}

- (NSString *)name {
    return _myName;
}

- (uint64_t)version {
    return _myVersion;
}

- (BOOL)migrateDatabase:(FMDatabase *)database error:(out NSError *__autoreleasing *)error {
    for (NSString *updateStr in _updateArray) {
        [database executeUpdate:updateStr];
    }
    return YES;
}

@end
```
```
// #define CACHESDIRECTORY ([NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) objectAtIndex:0])

    NSString *documentsPath = CACHESDIRECTORY; // [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    
    // File path
    
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"express.sqlite"];
    NSLog(@"filePath == %@",filePath);
    // Instantiate FMDataBase object
    
    _db = [FMDatabase databaseWithPath:filePath];
    
    [_db open];

    FMDBMigrationManager *manager = [FMDBMigrationManager  managerWithDatabaseAtPath:filePath migrationsBundle:[NSBundle mainBundle]];
    // Here, migration 1 means new
    Migration *migration_1 = [[Migration alloc] initWithName:@"Newly build table surface" andVersion:1 andExecuteUpdateArray:@[@"CREATE TABLE 'express' ('express_number' VARCHAR(255),'express_name' VARCHAR(255),'express_type' VARCHAR(255), 'express_date' VARCHAR(255),'express_message' VARCHAR(255), 'express_image' ARCHAR(255))"]];
    [manager addMigration:migration_1];
    // Add fields
    Migration * migration_2=[[Migration alloc]initWithName:@"USer Table new fields email" andVersion:2 andExecuteUpdateArray:@[@"alter table express add email text"]];
    [manager addMigration:migration_2];
    
    BOOL resultState = NO;
    NSError *error = nil;
    if (!manager.hasMigrationsTable) {
        resultState = [manager createMigrationsTable:&error];
    }
    resultState = [manager migrateDatabaseToVersion:UINT64_MAX progress:nil error:&error];
    NSLog(@"Has `schema_migrations` table?: %@", manager.hasMigrationsTable ? @"YES" : @"NO");
    NSLog(@"Origin Version: %llu", manager.originVersion);
    NSLog(@"Current version: %llu", manager.currentVersion);
    NSLog(@"All migrations: %@", manager.migrations);
    NSLog(@"Applied versions: %@", manager.appliedVersions);
    NSLog(@"Pending versions: %@", manager.pendingVersions);
    
    [_db close];
```

## Installation

FMDBMigrationManager is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'FMDBMigrationManager', :git => 'https://github.com/c-shen/FMDBMigrationManager.git'
```
**Note**: Fork of *c-shen* has no Release tag so it's impossible to run podspec validation using it. 
Use this for instead

```ruby
pod 'FMDBMigrationManager', :git => 'https://github.com/schmidt9/FMDBMigrationManager.git'
```

## Author

sen 

## License

FMDBMigrationManager is available under the MIT license. See the LICENSE file for more info.
