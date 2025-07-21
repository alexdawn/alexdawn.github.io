---
tags: [refactoring, deprecation, doctrine-orm]
categories: [php]
---

## How to deprecate properties that should be a relationship

On a legacy project I had a problem, the entire database schema was converted
via an automatic process to doctrine entities. This meant even foreign keys had become doctrine properties. Some of which had been used across the codebase. This caused an issue because if you add a doctrine relationship on the same column, the second could clear the first one being set:

```php
#[Entity]
class Post
{
    #[Id, Column]
    private int $id;

    #[Column]
    private int $userId; // our foreign key, automatically generated as a property not as a ManyToOne as it should be

    #[ManyToOne(targetEntity: User::class, inversedBy: 'posts')] // our newly created relationship, using doctrine in the way it is intended to be used
    private User $user;

    // following are the 4 bog standard getter/setters
    public function getUser(): User
    {
        return $this->user;
    }

    public function setUser(User $user): static
    {
        $this->user = $user;
        return $this;
    }

    public function getUserId(): int
    {
        return $this->userId;
    }

    public function setUserId(int $userId): static
    {
        $this->userId = $userId;
        return $this;
    }
}

// in usage
(new Post())
    ->setUserId($userId)
    ->setUser($user); // redundant to set both but if you don't doctrine will overwrite the first with the second, on legacy code it will be unset, and if not nullable it will error
```

the above would cause an even bigger problem in my situation as the entities were using autoincrement integers rather than a GUID as recommended by doctrine,
for example:

```php
(new User())
    ->setValues();

(new Post())
    ->setUserId($userId) // what should this be? the id is not yet known until flush time, but it cannot be left unset
    ->setUser($user);
```

The obvious solution is that $userId should no longer be a property then we can safely use `->setUser` in all circumstances, however when this entity is used across multiple apps it becomes a harder job to make such a breaking change all in one go. Instead we need a nice clean minor change with deprecating not removing the old getter/setter and use the `trigger_deprecation` function provided by `symfony/deprecation-contracts`:

```php
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ORM\HasLifecycleEvents]
class Post
{
    /** @deprecated no longer a property */
    private int $userId;

    #[ManyToOne(targetEntity: User::class, inversedBy: 'posts')]
    private User $user;

    ... // $user getter/setters as previously shown

    /** @deprecated */
    public function getUserId(): int
    {
        trigger_deprecation('org/project', 'x.y', 'replace getUserId with getUser()->getId()');
        return $this->user->getId();
    }

    /** @deprecated */
    public function setUserId(int $userId): static
    {
        trigger_deprecation('org/project', 'x.y', 'replace setUserId with setUser($userRepository->getReference($id))');
        $this->userId = $userId;
        return $this;
    }

    #[ORM\PrePersist]
    #[ORM\PreUpdate]
    public function handleDeprecatedUserIdProperty(EventArgs $event)
    {
        if (isset($this->userId)) {
            $user = $event->getObjectManager()->getRepository($this)->getReference($this->userId);
            $this->setUser($user);
            unset($this->userId);
        }
    }
}
```

You see the class must have the lifecycle event attribute/annotation for doctrine to call the callback method,
this provides a very useful `EventArgs` class to the entity, through this class it is then possible to obtain the entity manager and
even the repository that manages the entity itself. This is highly useful as a method to dependency inject these services. Since doctrine entities are not registered as symfony services (sensibly so as we want to be able to construct as many as we want and to dependency inject a shared and pre-constructed entity doesn't make sense in most circumstances) the normal methods of dependency injection via the constructor is not possible.

Via the above callback `handleDeprecatedUserIdProperty` even if userId is no longer a doctrine property we can make use of the php property to set the relationship to what it should be in a way the means even old code can continue to work as is, while new code able to make use of the better `getUser()` instead of the less useful `getUserId()`.